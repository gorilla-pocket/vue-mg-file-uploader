<template>
  <section class="container mt-2">
    <file-uploader
        :token="token"
        postAction="/api/attachments"
        :uploaded-files.sync="uploadedFiles"
    >
    </file-uploader>
    <hr>
    <file-uploader
        :token="token"
        postAction="/api/attachments"
        :uploaded-files.sync="uploadedFiles"
    >
      <template v-slot:select>
        添付ファイル
      </template>
    </file-uploader>
    <hr>
    <file-uploader
        :token="token"
        postAction="/api/attachments"
        :uploaded-files.sync="uploadedFiles"
        read-only
    >
    </file-uploader>
  </section>
</template>

<script>
import FileUploader from '../src/vue-mg-file-uploader'
import 'bootstrap/dist/css/bootstrap.min.css'
export default {
  data() {
    return {
      uploadedFiles: [],
      token: '',
    }
  },
  mounted: function () {
    this.uploadedFiles = [
      { id: 1, name: 'file01.txt', size: 2000},
      { id: 2, name: 'file02.txt', size: 12000},
    ]
  },
  methods: {
    //
  },
  components: {
    FileUploader,
  },  
}
</script>

<style scoped>
@import 'https://use.fontawesome.com/releases/v5.7.2/css/all.css';
</style>