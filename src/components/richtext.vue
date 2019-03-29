<template>
  <div class='rich-text'>
    <!-- 富文本编辑器 -->
    <Editor v-model='content'
            id='tinymce'
            :init='setting'
            @onInit='editorInit'
            @onKeyDown='editorKeyDown'></Editor>
  </div>
</template>

<script>
import Editor from '@tinymce/tinymce-vue';
import tinymce from 'tinymce/tinymce';
import 'tinymce/themes/modern/theme';
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
import 'tinymce/plugins/contextmenu';
import 'tinymce/plugins/directionality';
import 'tinymce/plugins/emoticons';
import 'tinymce/plugins/paste';
import 'tinymce/plugins/textcolor';
import 'tinymce/plugins/colorpicker';
import 'tinymce/plugins/textpattern';

import 'tinymce/plugins/codesample';

// import { uploadFile } from '@/api/user'

export default {
  components: {
    Editor
  },
  props: {
    url: {
      default: '',
      type: String
    },
    accept: {
      default: 'image/jpeg, image/png',
      type: String
    },
    maxSize: {
      default: 2097152,
      type: Number
    },
    withCredentials: {
      default: false,
      type: Boolean
    }
  },
  data () {
    let self = this;
    return {
      content: '',
      setting: {
        branding: false,
        elementpath: false,
        height: 300,
        language_url: '/static/langs/zh_CN.js',
        language: 'zh_CN',
        menubar: 'edit insert view format table tools',
        external_plugins: {
          'emoticons': '/static/emoticons/plugin.min.js',
          'codesample': '/static/codesample/plugin.min.js'
        },
        skin_url: '/skins/lightgray',
        plugins: [
          'advlist lists link image charmap hr anchor pagebreak imagetools',
          'searchreplace visualblocks visualchars code fullpage',
          'insertdatetime nonbreaking save table contextmenu directionality',
          'emoticons paste textcolor colorpicker textpattern codesample'
        ],
        toolbar: [
          ' newnote | undo redo | insert | styleselect | fontselect | formatselect | fontsizeselect | forecolor backcolor bold italic | alignleft aligncenter alignright alignjustify ',
          ' bullist numlist outdent indent | link image | codesample emoticons | table ',
        ],
        autosave_interval: '20s',
        forced_root_block: 'p',
        force_p_newlines: true,
        importcss_append: true,
        // CONFIG: ContentStyle 这块很重要， 在最后呈现的页面也要写入这个基本样式保证前后一致， `table`和`img`的问题基本就靠这个来填坑了
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
            styles: { 'text-indent': '2em' }
          },
          {
            title: '行高',
            items: [
              { title: '1', styles: { 'line-height': '1' }, inline: 'span' },
              {
                title: '1.5',
                styles: { 'line-height': '1.5' },
                inline: 'span'
              },
              { title: '2', styles: { 'line-height': '2' }, inline: 'span' },
              {
                title: '2.5',
                styles: { 'line-height': '2.5' },
                inline: 'span'
              },
              { title: '3', styles: { 'line-height': '3' }, inline: 'span' }
            ]
          }
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
        object_resizing: true,

        // Image
        imagetools_toolbar:
          'rotateleft rotateright | flipv fliph | editimage imageoptions',

        images_upload_handler: function (blobInfo, success, failure) {
          if (blobInfo.blob().size > self.maxSize) {
            failure('文件体积过大');
          }
          if (self.accept.indexOf(blobInfo.blob().type) >= 0) {
            uploadPic();
          } else {
            failure('图片格式错误');
          }
          async function uploadPic () {
            let formData = new FormData();
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
        }
      },

    };
  },
  created () {

  },
  mounted () {
    tinymce.init({});
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
};
</script>

<style lang='less'>
.rich-text {
  padding-left: 100px;
}
</style>
