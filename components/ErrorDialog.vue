<template>
  <v-row justify="center" v-if="errorDialogObj.show">


    <v-dialog
      max-width="500"
      v-model="errorDialogObj.show"
    >
      <v-card>
        <v-card-title class="headline"> {{errorDialogObj.title}}</v-card-title>

        <v-card-text>
          {{errorDialogObj.message}}
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            @click="$emit('closeErrorDialog')"
            outlined
            text
          >
            Conferma
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
  export default {
    props: {
      errorDialogObj: {
        type: Object
      },
    },
  }
</script>
