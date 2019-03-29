<template>
  <div class='rich-text'
       id='rich-text'>
    <!-- 富文本编辑器 -->
    <Editor v-model='orcontent'
            id='tinymce'
            :init='setting'
            @onInit='editorInit'
            @onKeyDown='editorKeyDown'></Editor>
  </div>
</template>

<script>
import Editor from '@tinymce/tinymce-vue';
import tinymce from 'tinymce/tinymce';
import 'tinymce/themes/silver/theme';
import 'tinymce/plugins/advlist';
import 'tinymce/plugins/lists';
import 'tinymce/plugins/link';
import 'tinymce/plugins/image';
import 'tinymce/plugins/charmap';
import 'tinymce/plugins/hr';
import 'tinymce/plugins/anchor';
import 'tinymce/plugins/pagebreak';
import 'tinymce/plugins/imagetools';
import 'tinymce/plugins/searchreplace';
import 'tinymce/plugins/visualblocks';
import 'tinymce/plugins/visualchars';
import 'tinymce/plugins/code';
import 'tinymce/plugins/fullpage';
import 'tinymce/plugins/insertdatetime';
// import 'tinymce/plugins/media';
import 'tinymce/plugins/nonbreaking';
import 'tinymce/plugins/save';
import 'tinymce/plugins/table';
// import 'tinymce/plugins/contextmenu';
import 'tinymce/plugins/directionality';
import 'tinymce/plugins/emoticons';
import 'tinymce/plugins/paste';
// import 'tinymce/plugins/textcolor';
// import 'tinymce/plugins/colorpicker';
import 'tinymce/plugins/textpattern';
import 'tinymce/plugins/codesample';
// import 'tinymce/plugins/autoresize';

export default {
  components: {
    Editor,
  },
  props: {
    accept: {
      default: 'image/jpeg, image/png',
      type: String,
    },
    maxSize: {
      default: 2097152,
      type: Number,
    },
  },
  data () {
    const self = this;
    return {
      orcontent: '',
      setting: {
        // 禁用产品属性状态栏中显示的“ Powered by Tiny ”
        branding: false,
        // 禁用调整大小
        resize: false,
        // 禁用element path编辑器底部的状态栏。
        // elementpath: true,
        // autoresize_on_init: true,

        height: 450,
        language_url: '/static/langs/zh_CN.js',
        language: 'zh_CN',
        menubar: 'edit insert view format table tools',
        external_plugins: {
          'emoticons': '/static/emoticons/plugin.min.js',
          'codesample': '/static/codesample/plugin.min.js',
        },
        skin_url: '/skins/ui/oxide',
        content_css: '/skins/content/default/content.min.css',
        // 插件
        plugins: [ // colorpicker textcolor contextmenu autoresize
          'advlist lists link image charmap hr anchor pagebreak imagetools',
          'searchreplace visualblocks visualchars code fullpage',
          'insertdatetime nonbreaking save table directionality',
          'emoticons paste textpattern codesample',
        ],
        // 工具栏控件
        toolbar: [
          ' newnote | undo redo | insert | styleselect | fontselect | formatselect | fontsizeselect | forecolor backcolor bold italic | alignleft aligncenter alignright alignjustify ',
          ' bullist numlist outdent indent | link image | codesample emoticons | table ',
        ],
        // 自动保存的时间
        // autosave_interval: '20s',
        // 确保任何非块元素或文本节点都包含在p块元素中
        forced_root_block: 'p',
        // 导入的样式附加到Format菜单的末尾，并将替换默认格式。
        importcss_append: true,
        // 使TinyMCE的可编辑区域具有与周围内容相同的样式， `table`和`img`的问题基本就靠这个来填坑了
        content_style: `
          *                         { padding:0; margin:0; }
          html, body                { height:100%; }
          img                       { max-width:100%; display:block;height:auto; }
          a                         { text-decoration: none; }
          iframe                    { width: 100%; }
          p                         { line-height:1.6; margin: 0px; }
          table                     { word-wrap:break-word; word-break:break-all; max-width:100%; border:none; border-color:#999; }
          .mce-object-iframe        { width:100%; box-sizing:border-box; margin:0; padding:0; }
          ul,ol                     { list-style-position:inside; }
        `,
        insert_button_items: 'image link | inserttable',
        // CONFIG: Paste
        paste_retain_style_properties: 'all',
        paste_word_valid_elements: '*[*]', // word需要它
        paste_data_images: true, // 粘贴的同时能把内容里的图片自动上传，非常强力的功能
        paste_convert_word_fake_lists: false, // 插入word文档需要该属性
        paste_webkit_styles: 'all',
        paste_merge_formats: true,
        nonbreaking_force_tab: false,
        paste_auto_cleanup_on_paste: false,
        // CONFIG: StyleSelect
        style_formats: [
          {
            title: '首行缩进',
            block: 'p',
            styles: { 'text-indent': '2em' },
          },
          {
            title: '行高',
            items: [
              { title: '1', styles: { 'line-height': '1' }, inline: 'span' },
              {
                title: '1.5',
                styles: { 'line-height': '1.5' },
                inline: 'span',
              },
              { title: '2', styles: { 'line-height': '2' }, inline: 'span' },
              {
                title: '2.5',
                styles: { 'line-height': '2.5' },
                inline: 'span',
              },
              { title: '3', styles: { 'line-height': '3' }, inline: 'span' },
            ],
          },
        ],
        // FontSelect
        font_formats: `
          微软雅黑=微软雅黑;
          宋体=宋体;
          黑体=黑体;
          仿宋=仿宋;
          楷体=楷体;
          隶书=隶书;
          幼圆=幼圆;
          Andale Mono=andale mono,times;
          Arial=arial, helvetica,
          sans-serif;
          Arial Black=arial black, avant garde;
          Book Antiqua=book antiqua,palatino;
          Comic Sans MS=comic sans ms,sans-serif;
          Courier New=courier new,courier;
          Georgia=georgia,palatino;
          Helvetica=helvetica;
          Impact=impact,chicago;
          Symbol=symbol;
          Tahoma=tahoma,arial,helvetica,sans-serif;
          Terminal=terminal,monaco;
          Times New Roman=times new roman,times;
          Trebuchet MS=trebuchet ms,geneva;
          Verdana=verdana,geneva;
          Webdings=webdings;
          Wingdings=wingdings,zapf dingbats`,
        // Tab
        tabfocus_elements: ':prev,:next',
        // 允许您调整表格和图像的大小
        object_resizing: true,
        // Image
        // imagetools_toolbar: 'rotateleft rotateright | flipv fliph | editimage imageoptions',
        images_upload_handler (blobInfo, success, failure) {
          async function uploadPic () {
            const formData = new FormData();
            // 服务端接收文件的参数名，文件数据，文件名
            formData.append('upfile', blobInfo.blob(), blobInfo.filename());
            // 图片上传请求
            /* const res = await uploadFile(formData);
            if (res.success) {
              success(res.data[0].FileUrl);
            } else {
              failure('上传失败');
            } */
          }
          if (blobInfo.blob().size > self.maxSize) {
            failure('文件体积过大');
          }
          if (self.accept.indexOf(blobInfo.blob().type) >= 0) {
            uploadPic();
          } else {
            failure('图片格式错误');
          }
        },
      },
    };
  },
  watch: {
    orcontent (newVal) {
      localStorage.richContent = newVal;
    },
  },
  methods: {
    // 初始化编辑器
    editorInit () {

    },
    // 保存文案到本地
    editorKeyDown () {
      // localStorage.editorContent = this.content;
    },
  },
  created () {
    // 组件初始化时清空content
    // localStorage.richContent = '';
    // 如果是修改产品，则orcontent有初始值
    try {
      const product = JSON.parse(localStorage.product);
      this.orcontent = product.DescriptionContent;
    } catch (error) {

    }
  },
  mounted () {
    tinymce.init({});
    // document.body.appendChild(this.$el);
  },
  destroyed () {
    // localStorage.richContent = '';
  },
};
</script>

<style lang='less'>
</style>
