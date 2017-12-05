<template>
  <div id="app">
    <header>
      <nav>
        <div class="nav-wrapper blue-grey darken-2">
          <div class="container">
            <a href="javascript:void(0)" @click="reset()" class="brand-logo ">RUDIZIP</a>
            <ul id="nav-mobile" class="right hide-on-med-and-down">
              <li><a href="javascript:void(0)" @click="reset()">Comprimir</a></li>
              <li><a href="javascript:void(0)" @click="reset_about()">Sobre</li>
              </ul>
            </div>
          </div>
        </nav>
      </header>
      <main>
        <div class="container">
          <!--UPLOAD-->
          <form enctype="multipart/form-data" novalidate v-if="isInitial || isSaving">
              <div class="container center tryitout"><h4>Envie uma imagem!</h4></container>
            <div class="dropbox">
              <input type="file" multiple :name="uploadFieldName" :disabled="isSaving" @change="filesChange($event.target.name, $event.target.files); fileCount = $event.target.files.length"
              accept="image/*" class="input-file">
              <p v-if="isInitial">
                Arraste um arquivo aqui,<br> ou clique para procurar! 
              </p>
              <p v-if="isSaving">
                Enviando {{ fileCount }} arquivo...
              </p>
            </div>
          </form>
          <!--SUCCESS-->
          <div v-if="isSuccess">
            <div  class="row">
              <div v-for="item in uploadedFiles.slice(0,2)" class="col s6 m6">
                <div class="card">
                  <div class="card-image">
                    <img :src="item.url" class="img-responsive " :alt="item.originalName">
                    <span class="card-title">{{item.originalName}}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <!--FAILED-->
          <div v-if="isFailed">
            <h2>Uploaded failed.</h2>
            <p>
              <a href="javascript:void(0)" @click="reset()">Try again</a>
            </p>
        <pre>{{ uploadError }}</pre>
          </div>
          <div v-if="isAbout">
            <div class="container">
              <p class="flow-text">
                Este trabalho serve para demonstrar que o aluno teve aproveitamento da matéria multimída/hipermídia.<br>
                O backend foi feito em <a href="https://nodejs.org">NodeJS</a>, o compressor em linguagem C e o front em <a href="https://vuejs.org/">VueJS</a> e <a href="http://materializecss.com/">Materialize</a>. <br> 
                Para o compressor, o desenvolvimento se deu por <a href="https://pt.wikipedia.org/wiki/Test_Driven_Development">TDD (Test Driven Devolpment)</a> com a ajuda da biblioteca <a href="https://github.com/Snaipe/Criterion">Criterion</a>.
              </p>
            </div>
          </div>
        </div>
      </main>
      <footer class="page-footer blue" style="position:fixed;bottom:0;left:0;width:100%;">
        <div class="container">
          <div class="row">
            <div class="col l6 s12">
              <h5 class="white-text">Multimídia e Hipermídia!</h5>
              <p class="grey-text text-lighten-4">Trabalho de conclusão da disciplina Multimídia/Hipermídia.</p>
            </div>
            <div class="col l4 offset-l2 s12">
              <h5 class="white-text">Links</h5>
              <ul>
                <li><a class="grey-text text-lighten-3" href="https://nodejs.org">NodeJS</a></li>
                <li><a class="grey-text text-lighten-3" href="http://materializecss.com/">Materialize</a></li>
                <li><a class="grey-text text-lighten-3" href="https://github.com/Snaipe/Criterion">Criterion</a></li>
                <li><a class="grey-text text-lighten-3" href="https://vuejs.org/">VueJS</a></li>
              </ul>
            </div>
          </div>
        </div>
        <div class="footer-copyright">
          <div class="container">
           Abbudcorp  © 2017 
            <a class="grey-text text-lighten-4 right" href="#!">USP/ICMC/EESC</a>
          </div>
        </div>
      </footer>
    </div>
  </template>

  <script>
// swap as you need
  //import { upload } from './file-upload.fake.service'; // fake service
import { upload } from './file-upload.service';   // real service
import { wait } from './utils';
const STATUS_INITIAL = 0, STATUS_SAVING = 1, STATUS_SUCCESS = 2, STATUS_FAILED = 3,STATUS_ABOUT= 4;
const BASE_URL = 'http://localhost:3001';
export default {
  name: 'app',
  data() {
    return {
      uploadedFiles: [],
      uploadError: null,
      currentStatus: null,
      uploadFieldName: 'photos'
    }
  },
  computed: {
    isInitial() {
      return this.currentStatus === STATUS_INITIAL;
    },
    isSaving() {
      return this.currentStatus === STATUS_SAVING;
    },
    isSuccess() {
      return this.currentStatus === STATUS_SUCCESS;
    },
    isFailed() {
      return this.currentStatus === STATUS_FAILED;
    },
    isAbout() {
      return this.currentStatus === STATUS_ABOUT;
    }
  },
  methods: {
    reset_about() {
      // reset form to initial state
      this.currentStatus = STATUS_ABOUT;
      this.uploadedFiles = [];
      this.uploadError = null;
    },
    reset() {
      // reset form to initial state
      this.currentStatus = STATUS_INITIAL;
      this.uploadedFiles = [];
      this.uploadError = null;
    },
    save(formData) {
      // upload data to the server
      this.currentStatus = STATUS_SAVING;
      const url = `${BASE_URL}/photos/upload`;

      upload(formData)
        .then(wait(1500)) // DEV ONLY: wait for 1.5s 
        .then(x => {
          this.uploadedFiles = [].concat(x);
          this.currentStatus = STATUS_SUCCESS;
        })
        .catch(err => {
          this.uploadError = err.response;
          this.currentStatus = STATUS_FAILED;
        });
    },
    filesChange(fieldName, fileList) {
      // handle file changes
      const formData = new FormData();

      if (!fileList.length) return;

      // append the files to FormData
      Array
        .from(Array(fileList.length).keys())
        .map(x => {
          formData.append(fieldName, fileList[x], fileList[x].name);
        });

      // save it
      this.save(formData);
    }
  },
  mounted() {
    this.reset();
  },
}

</script>

<style lang="scss">
.dropbox {
  outline: 2px dashed dodgerblue; /* the dash box */
  outline-offset: -2px;
  background: white;
  color: dimgray;
  padding: 10px 10px;
  min-height: 200px; /* minimum height */
  position: relative;
  cursor: pointer;
}
.tryitout{
  color :dodgerblue;
}

.input-file {
  opacity: 0; /* invisible but it's there! */
  width: 100%;
  height: 200px;
  position: absolute;
  cursor: pointer;
}

.dropbox:hover {
  background: dodgerblue; /* when mouse over to the drop zone, change color */
  color:lavender; /* when mouse over to the drop zone, change color */
}

.dropbox p {
  font-size: 1.2em;
  text-align: center;
  padding: 50px 0;
}
.body {
  display: flex;
  min-height: 100vh;
  flex-direction: column;
}
.main {
  flex: 1 0 auto;
}
.brand-logo{
    font-family: 'Anton',sans-serif;
}
</style>
