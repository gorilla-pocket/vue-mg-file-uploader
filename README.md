# vue-mg-file-uploader
vue-mg-file-uploader

![npm](https://img.shields.io/npm/v/vue-mg-file-uploader)
![npm](https://img.shields.io/npm/dm/vue-mg-file-uploader)

## Example

![image](https://user-images.githubusercontent.com/52206492/93536083-401e9700-f983-11ea-8195-bbceabdb92a6.png)

## Installation

```
npm i vue-mg-file-uploader
```

## Usage

app.js

```javascript
import FileUploader from 'vue-mg-file-uploader'
Vue.component('FileUploader', FileUploader)
```

Example:

```html
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
        use_description
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
      { id: 1, name: 'file01.txt', size: 2000, description: 'ああああああ'},
      { id: 2, name: 'file02bbbbbbbbbbbbbbbbbbbbbbbbb.txt', size: 12000},
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
```

## License

MIT
