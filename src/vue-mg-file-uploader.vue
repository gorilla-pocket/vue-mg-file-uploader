<template>
<div class="vue-mg-file-uploader">
    <div class="example-simple">
        <div class="upload">
            <div class="example-btn" v-if="!readOnly">
                <file-upload
                    class="btn btn-primary"
                    :post-action="postAction"
                    :extensions="extensions"
                    :accept="accept"
                    :multiple="true"
                    :size="file_size"
                    v-model="files"
                    :headers="{'X-CSRF-TOKEN': token}"
                    @input-filter="inputFilter"
                    @input-file="inputFile"
                    ref="upload">
                    <slot name="select">
                        <i class="fa fa-plus"></i> ファイル選択
                    </slot>
                </file-upload>
                <template v-if="!auto_start">
                    <button type="button" class="btn btn-success" v-if="!$refs.upload || !$refs.upload.active" @click.prevent="$refs.upload.active = true">
                        <i class="fa fa-arrow-up" aria-hidden="true"></i> Start Upload
                    </button>
                    <button type="button" class="btn btn-danger"  v-else @click.prevent="$refs.upload.active = false">
                        <i class="fa fa-stop" aria-hidden="true"></i> Stop Upload
                    </button>
                </template>
                <span class="text-muted ml-2">(サイズの上限: {{file_size | formatSizeLg}})</span>
            </div>
            <ul v-if="files.length||filter_local_updated_files.length">
                <li v-for="file in filter_local_updated_files" :key="file.id">
                    <a href="" @click.prevent="onDownload(file)">
                        {{file.name}}
                    </a>
                    <span class="ml-2">({{file.size | formatSize}})</span>
                    <button type="button" class="remove-upload remove-uploaded btn btn-sm btn-outline-danger ml-2" style="display:inline-block;" v-if="!readOnly" @click.prevent="removeUploadedFile(file)">
                        <i class="fas fa-trash-alt"></i>
                    </button>
                </li>
                <li v-for="file in files" :key="file.id">
                    <!-- <span><img class="thumb" v-bind:src="file.blob"></span> -->
                    <span>{{file.name}}</span>
                    <span class="ml-2">({{file.size | formatSize}})</span> -
                    <span class="text-danger" v-if="file.error">{{error_message(file.error)}}</span>
                    <span class="text-success" v-else-if="file.success">success</span>
                    <!-- <span class="text-info" v-else-if="file.active">active</span> -->
                    <!-- <span v-else></span><i class="fas fa-times"></i></span> -->
                    <div class="progress ml-2" style="display:inline-block;width:10rem;" v-if="file.active || file.progress !== '0.00'">
                        <div :class="{'progress-bar': true, 'progress-bar-striped': true, 'bg-danger': file.error, 'progress-bar-animated': file.active}" role="progressbar" :style="{width: file.progress + '%'}">{{file.progress}}%</div>
                    </div>
                    <button type="button" class="remove-upload remove-uploaded btn btn-sm btn-outline-danger ml-2" style="display:inline-block;" v-if="!file.error" @click.prevent="remove(file)">
                        <i class="fas fa-trash-alt"></i>
                    </button>
                </li>
            </ul>
        </div>
    </div>
</div>
</template>

<script>
import axios from 'axios'
import FileUpload from 'vue-upload-component'
export default {
    props: {
        token: {
            type: String,
        },
        postAction: {
            type: String,
        },
        extensions: {
            default: Array,
        },
        accept: {
            type: String,
        },
        size: {
            type: Number,
            default: 0,
        },
        auto_start: {
            default: true,
        },
        uploadedFiles: {
            default: Array,
        },
        readOnly: {
            type: Boolean,
            default:  false,
        }
    },
    data: function() {
        return {
            files: [],
            // local_files: [],
            local_updated_files: [],
        }
    },
    mounted: function () {
        this.local_updated_files = this.uploadedFiles
    },
    filters: {
        formatSize: function (val) {
            return (val/1024).toLocaleString() + ' KB';
        },
        formatSizeLg: function (val) {
            return (Math.ceil(val/1024/1024)).toLocaleString() + ' MB';
        }
    },
    watch: {
        uploadedFiles: function (val) {
            this.local_updated_files = val
        }
    },
    computed: {
        file_size: function () {
            return this.size ? this.size : 1024 * 1024 * 5
        },
        error_message: function () {
            return function (error) {
                switch (error) {
                    case 'size':
                        return 'ファイルサイズが大きすぎます。'
                    default:
                        return 'エラーが発生しました。('+error+')'
                }
            }
        },
        filter_local_updated_files: function () {
            return this.local_updated_files.filter((elem) => !elem.not_visibled)
        }
    },
    methods: {
        inputFile: function (newFile, oldFile) {
            if (newFile && oldFile && !newFile.active && oldFile.active) {
                // console.log('response', newFile.response)
                if (newFile.xhr) {
                    // console.log('status', newFile.xhr.status)
                    // console.log('file', newFile)
                    // this.local_updated_files.push({
                    //     // id: newFile.response.file.id,
                    //     name: newFile.name,
                    //     size: newFile.size,
                    //     file_id: newFile.id,
                    //     not_visibled: false,
                    // })
                    const f = {
                        id: newFile.response.file.id,
                        name: newFile.name,
                        size: newFile.size,
                        file_id: newFile.id,
                        not_visibled: true,
                    }
                    if (!newFile.error) {
                        const index = this.files.findIndex( (file) => file.id == newFile.id)
                        if (index >= 0) {
                            this.files.splice(index, 1)
                            f.not_visibled = false
                        }
                    }
                    this.local_updated_files.push(f)
                    this.$emit('update:uploadedFiles', this.local_updated_files)
                }
            }
            if (!newFile && oldFile) {
                // remove
                if (oldFile.success && oldFile.response.id) {
                // $.ajax({
                //   type: 'DELETE',
                //   url: '/upload/delete?id=' + oldFile.response.id,
                // })
                }
                const file = this.local_updated_files.findIndex((elem) => elem.file_id === oldFile.id)
                if (file && file.id) {
                    const _this = this
                    axios.delete('/api/attachments/'+file.id)
                    .then(function () {
                        const index = _this.local_updated_files.findIndex((elem) => elem.file_id === oldFile.id)
                        if (index !== -1) {
                            _this.local_updated_files.splice(index, 1)
                            _this.$emit('update:uploadedFiles', _this.local_updated_files)
                        }
                    })
                    // .catch(function (resp) {
                    //     // console.log(resp)
                    // })
                    // .finally(function () {
                    //     //
                    // })
                }
            }
            
            // Automatically activate upload
            if (Boolean(newFile) !== Boolean(oldFile) || oldFile.error !== newFile.error) {
                if (!this.$refs.upload.active) {
                    this.$refs.upload.active = true
                }
            }
        },
        inputFilter: function (newFile, oldFile, prevent) {
            if (newFile && !oldFile) {
                // Before adding a file

                // if (!/\.(jpeg|jpe|jpg|gif|png|webp)$/i.test(newFile.name)) {
                //     return prevent()
                // }
                const i = 0
                if (i) {
                    return prevent()
                }
            }
            // console.log(newFile)
            // newFile.blob = ''
            // let URL = window.URL || window.webkitURL
            // if (URL && URL.createObjectURL) {
            //     newFile.blob = URL.createObjectURL(newFile.file)
            // }
        },
        remove: function (file) {
            if (!confirm('削除します。よろしいですか？')) return
            this.$refs.upload.remove(file)
        },
        onDownload: function (file) {
            // const f = this.local_files.find((elem) => elem.file_id === file.id)
            // if (f) return
            axios({
                method:'post',
                url:'/api/attachments/download/'+file.id,
                responseType:'blob',
                dataType: 'binary',
            })
            .then(function (res1) {
                const url = window.URL.createObjectURL(new Blob([res1.data]))
                const link = document.createElement('a')
                link.href = url
                link.setAttribute('download', file.name)
                document.body.appendChild(link)
                link.click()
            });
        },
        removeUploadedFile: function (file) {
            if (!confirm('削除します。よろしいですか？')) return
            const _this = this
            axios.delete('/api/attachments/'+file.id)
            .then(function () {
                const index = _this.local_updated_files.findIndex((elem) => elem.id === file.id)
                if (index !== -1) {
                    _this.local_updated_files.splice(index, 1)
                    _this.$emit('update:uploadedFiles', _this.local_updated_files)
                }
            })
            // .catch(function (resp) {
            //     // console.log(resp)
            // })
            // .finally(function () {
            //     //
            // })
        },
    },
    components: {
        FileUpload,
    }
}
</script>

<style scoped>
.file-uploads {
  overflow: hidden;
  position: relative;
  text-align: center;
  display: inline-block;
}
.remove-upload {
    border: none;
}
ul {
  padding: .5rem 1.5rem;
  margin-bottom: 0;
}
.thumb {
  width: 50px;
}
.btn-link {
  padding: 0;
}
</style>