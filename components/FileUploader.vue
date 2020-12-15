<template>
  <v-row align-content="center" justify="center">
    <v-col align="center" cols=8>
      <v-card class="pt-1">
        <v-card-title class="justify-center ma-3">
          <ul v-if="files.length > 0">
            <slot></slot>
            <li>
              <span>{{files[0].name}}</span> -
              <span>{{files[0].size | formatSize}}</span> -
              <span v-if="files[0].error">{{files[0].error}}</span>
              <span v-else-if="files[0].success">success</span>
              <span v-else-if="files[0].active">active</span>
              <span v-else></span>
            </li>
          </ul>
          <ul v-else>
            <td colspan="7">
              <slot></slot>
              <div class="text-center p-5">
                <h5>Carica un file di tipo xls/xlsx</h5>
              </div>
            </td>
          </ul>
        </v-card-title>
        <v-divider></v-divider>

        <div class="example-drag">
          <div class="drop-active" v-show="$refs.upload && $refs.upload.dropActive">
            <h3>Drop files to upload</h3>
          </div>
        </div>

        <div class="example-btn">
          <v-btn class="my-5 mr-5" color="primary" outlined>
            <file-upload
              :accept="formatAccepted"
              :drop="true"
              :drop-directory="false"
              :maximum="1"
              ref="upload"
              v-model="files">
              <v-icon>mdi-plus</v-icon>
              Seleziona file
            </file-upload>
          </v-btn>
          <v-btn :disabled="files.length == 0"
                 @click="uploadFiles"
                 @click.prevent="$refs.upload.active = true"
                 color="#B42684"
                 outlined
                 v-if="!$refs.upload || !$refs.upload.active">
            <v-icon>mdi-arrow-up</v-icon>
            Carica file
          </v-btn>
          <v-btn @click.prevent="$refs.upload.active = false"
                 color="primary"
                 outlined
                 v-else>
            <v-icon>mdi-stop</v-icon>
            Stop Upload
          </v-btn>
        </div>
        <error-dialog :error-dialog-obj="errorDialogObj" @closeErrorDialog="errorDialogObj.show = false"/>
      </v-card>
    </v-col>
    <v-snackbar
      :color="snackbarColor"
      :timeout="3000"
      :top="true"
      v-model="openSnackbar"
    >
      {{ snackbarText }}
    </v-snackbar>
  </v-row>

</template>


<script>
  import FileUpload from 'vue-upload-component'
  import xlsxParser from 'xlsx-parse-json';
  import ErrorDialog from "~/components/ErrorDialog";
  import _ from 'lodash';

  // .csv, .xls, .xlsx
  export default {
    components: {
      FileUpload,
      ErrorDialog
    },
    props: ['formatAccepted', 'endPoint', 'accessToken', 'isAuthenticatedAndAuthorizedUser','fileStructure'],
    data: () => ({
      files: [],
      openSnackbar: false,
      snackbarText: '',
      snackbarColor: 'error',
      errorDialogObj: {
        show: false,
        title: 'Errore caricamento file',
        message: 'messaggio errore'
      }
    }),
    filters: {
      formatSize: (size) => {
        return (size / 1024).toFixed(2) + ' KB'
      }
    },
    methods: {
      async uploadFiles() {
        for (let file of this.files) {
          if (this.checkExtension(file)) {
            this.files.splice(0) // reactive way to empty the array
            await this.sendFile(file)
          } else {
            this.errorMessage('Estensione del file caricato invalido')
          }
        }
      },

      checkExtension(file) {
        let fileExt = file.name;
        console.log('check exstension')
        fileExt = fileExt.substring(fileExt.lastIndexOf('.'));
        return this.$props.formatAccepted.indexOf(fileExt) > -1
      },

      async checkFileContent(file) {
        let fileStructure = _.cloneDeep(this.$props.fileStructure)
        let errorMessage = []
        await xlsxParser
          .onFileSelection(file)
          .then(data => {
            Object.keys(data).forEach(sheet => {
              data[sheet].forEach((row, index) => {
                fileStructure.forEach(({header, errorList}) => {
                  if (!row[header.name] || isNaN(+row[header.name])) {
                    errorList.push(index + 2)
                  }
                })
              })
            })
          });
        fileStructure.forEach(({header, errorList}) => {
          if (errorList.length > 0) errorMessage.push(`${header.name} mancante o formato errato sulla cella: ${errorList.map(row => `${header.col}:${row}`)}`)
        })
        if (errorMessage) {
          console.log(errorMessage.join('\n'));
          return errorMessage.join('\n')
        }
      },

      async sendFile(file) {
        try {
          if (this.$props.isAuthenticatedAndAuthorizedUser) {

            let formData = new FormData();
            formData.append('file', file.file);
            let errorMessage = await this.checkFileContent(file.file)
            if (errorMessage) {
              this.errorDialogObj.message = errorMessage
              this.errorDialogObj.show = true
            } else {
              const response = await this.$axios.post(this.$props.endPoint, formData,
                {
                  headers: {
                    'Content-Type': 'multipart/form-data',
                    'authorization': 'Bearer ' + this.$props.accessToken,
                  }
                })
              console.log(response.data)
              if (response.data) {
                this.successMessage()
              } else {
                this.errorMessage('Errore del caricamento del file, accertarsi che i campi utilizzati corrispondcno a quelli richiesti')
              }
            }
          }
        } catch (e) {
          console.log(e)
          this.errorMessage('Errore del caricamento del file, accertarsi che i campi utilizzati corrispondcno a quelli richiesti')
        }
      },
      successMessage() {
        this.snackbarText = 'Caricamento avvenuto con successo'
        this.snackbarColor = '#B42684'
        this.openSnackbar = true
      },
      errorMessage(text) {
        this.snackbarText = text
        this.snackbarColor = 'error'
        this.openSnackbar = true
      }
    },
  }
</script>
