<template>
  <div>
    <p class="export-pdf-btn" id="export-pdf" v-on:click="exportPDF('pdfResume')">导出PDF</p>
    <div class="page" id="pdfResume" ref="pdfResume">
      <section class="panel personal">
        <div class="col-4">
          <p class="name">{{ personal.name }}</p>
          <p class="job">{{personal.job}}</p>
        </div>
        <div class="col-8">
          <ul class="info">
            <li v-for="item in personal.info" :key="item.id">
              {{item.label}} : {{item.value}}
            </li>
          </ul>
        </div>
      </section>
      <section class="panel skill">
        <h2 class="panel-title">职业技能</h2>
        <p class="skill-item" v-for="item in skill" :key="item.index">{{item}}</p>
      </section>
      <section class="panel experience">
        <h2 class="panel-title">工作经验</h2>
        <div class="panel-content">
          <ul>
            <li class="experience-item " v-for="item in experience" :key="item.id">
              <div class="conpany-panel">
                <span class="company-name">{{item.company}} </span>
                <span class="job-time">{{item.timeStart}} - {{item.timeEnd}}</span>
              </div>
              <ul>
                <li class="project-item" v-for="pro in item.project" :key="pro.index">
                  <p class="project-name">{{pro.name}} <a class="project-link" v-if="pro.url" :href="pro.url"
                      target="_blank">{{pro.url}}</a></p>
                  <p class="project-intro" v-if="pro.intro"><span class="bold">项目简介 : </span>{{pro.intro}}</p>
                  <p>
                    <span class="bold">使用 : </span>{{pro.use}}</p>
                  <div class="project-details">
                    <span class="bold">主要职责 : </span>
                    <p v-for="detail in pro.details" :key="detail.index">{{detail}}</p>
                  </div>
                </li>
              </ul>
            </li>
          </ul>
        </div>
      </section>
      <section class="panel skill">
        <h2 class="panel-title">自我评价</h2>
        <p class="skill-item" v-for="item in evaluation" :key="item.index">{{item}}</p>
      </section>
      <section class="panel">
        <h2 class="panel-title">工作期望</h2>
        <div class="panel-content">
          <p><span class="bold">岗位 : </span>{{expectation.post}}</p>
          <p><span class="bold">地点 : </span>{{expectation.workplace}}</p>
          <p><span class="bold">薪资 : </span>{{expectation.salary}}</p>
        </div>
      </section>
    </div>
  </div>

</template>

<script type="text/javascript">
import html2Canvas from 'html2Canvas';
import JsPDF from 'jspdf';

export default {
  data() {
    return {
      personal: {
        name: '',
        job: '',
      }, // 个人信息
      skill: [], // 职业技能
      experience: [], // 工作经验
      evaluation: [], // 自我评价
      expectation: {}, // 工作期望
    };
  },
  components: {},
  created() {
    this.getData();
  },
  methods: {
    exportPDF(idSelect) {
      const pdfDom = document.getElementById(idSelect);
      pdfDom.style.background = '#FFFFFF'; // 设置背景色
      const c = document.createElement('canvas');
      // 导出来的pdf打印出来还是比较模糊的
      let opts = {
        scale: 2, //
        canvas: c,
        logging: true,
        width: pdfDom.width,
        height: pdfDom.height,
        useCORS: true, // 配置:开启跨域
        taintTest: true, // 在渲染器测试图片
        allowTaint: true, // 允许加载跨域的图片
      };

      c.width = pdfDom.width;
      c.height = pdfDom.height;
      c.getContext('2d').scale(2, 2);
      window.html2Canvas = html2Canvas;
      html2Canvas(pdfDom, opts).then((canvas) => {
        // 开始canvas截图
        // 开始准备工作
        const contentWidth = canvas.width;
        const contentHeight = canvas.height;// 内容为画布宽高
        const pageHeight = (contentWidth / 592.28) * 841.89;
        let leftHeight = contentHeight;
        let position = 0;
        // 此处图片格式可以是PNG，也可是JPEG，注意：需要考虑Browser支持的图片格式
        const imgWidth = 592.28;
        const imgHeight = (592.28 / contentWidth) * contentHeight;
        const context = canvas.getContext('2d');
        // 然后将画布缩放，将图像放大两倍画到画布上
        context.scale(2, 2);
        const pageData = canvas.toDataURL('image/jpeg', 1); // 将图片转为 base64
        pdfDom.style.display = 'none'; // 开始将图片转换为PDF // 设置纸张大小，方向
        const PDF = new JsPDF('', 'pt', 'a4');
        if (leftHeight < pageHeight) { // 如果内容高度小与一页纸的高度-单页
          PDF.addImage(pageData, 'jpeg', 0, 0, imgWidth, imgHeight);
        } else {
          while (leftHeight > 0) { // 多页
            PDF.addImage(pageData, 'jpeg', 0, position, imgWidth, imgHeight);
            leftHeight -= pageHeight;
            position -= 841.89;
            if (leftHeight > 0)PDF.addPage();
          }
        }// 保存PDF
        const pdfName = `${this.personal.name}-${this.personal.job}.pdf`;
        PDF.save(pdfName);
      });
    },
    getData() {
      this.$axios.get('/static/data/Demo.json').then((res) => {
        this.personal = res.data.personal;
        this.skill = res.data.skill;
        this.experience = res.data.experience;
        this.evaluation = res.data.evaluation;
        this.expectation = res.data.expectation;
      }).catch((error) => {
        throw new Error([error]);
      });
    },
  },
};

</script>

<style scoped lang="scss">
  @import "./src/assets/scss/color.scss";
  @import "./src/assets/scss/common.scss";
  @import "./src/assets/scss/sass-base.scss";

  .export-pdf-btn {
    position: absolute;
    top: 0;
    right: 20px;
    margin: 10px;
    width: 80px;
    height: 20px;
    font-size: 16px;
    line-height: 20px;
    border-radius: 10px;
    color: #2c3e50;
    border: 2px solid #2c3e50;
    background-color: #fff;
    cursor: pointer;

    &:hover {
      color: $primaryC;
      border: 2px solid $primaryC;
    }
  }

  .page {
    width: 100%;
    max-width: 600px;
    line-height: 1.4;
    font-size: 12px;
    margin: 0 auto;
    padding: 10px;

    .panel {
      box-sizing: border-box;
      padding: 5px 5px 0 10px;
      width: 100%;
      height: auto;
      text-align: left;
      @include clear;

      .panel-title {
        margin-bottom: 5px;
        font-size: 16px;
        line-height: 24px;
        font-weight: bold;
        color: $primaryC;
        border-bottom: 1px solid rgba(0, 0, 0, 0.1);
      }
    }
  }

  .personal {
    width: 100%;
    height: 100%;
    color: #fff;
    background-color: $primaryC;
    text-align: center;
    padding: 5px 10px !important;

    .name {
      width: 100%;
      font-size: 30px;
      line-height: 40px;
      font-weight: bold;
    }

    .job {
      font-size: 20px;
      font-weight: bold;
    }

    .info {
      font-size: 14px;
      line-height: 22px;
    }
  }

  .skill {
    .skill-item {
      position: relative;
      padding-left: 14px;

      &::before {
        position: absolute;
        top: 8px;
        left: 0;
        width: 6px;
        height: 6px;
        content: "";
        margin-right: 10px;
        font-weight: bold;
        background-color: $primaryC;
        border-radius: 3px;
      }
    }
  }

  .experience {
    width: 100%;

    .experience-item {
      border-left: 2px solid $primaryC;
      padding: 5px 0 0 20px;
      @include clear();

      .company-name {
        position: relative;
        width: 100%;
        height: auto;
        font-size: 15px;
        line-height: 24px;
        font-weight: bold;
        color: $primaryC;

        &::before {
          position: absolute;
          top: 5px;
          left: -27px;
          content: " ";
          width: 12px;
          height: 12px;
          border-radius: 6px;
          background-color: $primaryC;
        }
      }

      .job-time {
        color: $primaryC;
        font-size: 14px;
        line-height: 24px;
        font-weight: bold;
      }

      .project-item {
        margin-top: 5px;

        &:first-child {
          margin: 0;
        }

        .project-name {
          position: relative;
          font-size: 14px;
          line-height: 24px;
          max-height: 36px;
          font-weight: bold;
          padding-left: 30px;

          &::before {
            position: absolute;
            top: 0;
            left: 0;
            width: 6px;
            height: 6px;
            content: ">>";
            margin-right: 10px;
            font-size: 16px;
            font-weight: 700;
            color: $primaryC;
            border-radius: 3px;
          }

          .project-link {
            text-decoration: none;
            color: $primaryC;
          }

        }

        .project-details {
          p {
            padding-left: 14px;
            position: relative;

          }

          p::before {
            position: absolute;
            top: 8px;
            left: 0;
            width: 6px;
            height: 6px;
            content: "";
            margin-right: 10px;
            font-weight: bold;
            background-color: $primaryC;
            border-radius: 3px;
          }
        }
      }

    }
  }

</style>
