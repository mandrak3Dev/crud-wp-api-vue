<template>
  <v-data-table
    :headers="headers"
    :items="entradas"
    sort-by="id"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat color="white">
        <v-toolbar-title>My CRUD</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >New Item</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-text-field v-model="editedItem.title" label="Titulo"></v-text-field>
                  </v-col>
                  <!-- <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.id" label="id"></v-text-field>
                  </v-col> -->
                  <v-col cols="12">
                    <v-text-field type="text" v-model="editedItem.content" label="Contenido"></v-text-field>
                  </v-col>
                  <!-- <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.date" label="date (g)"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.status" label="status (g)"></v-text-field>
                  </v-col> -->
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">Reset</v-btn>
    </template>
  </v-data-table>
</template>


<script>
  export default {
    data: () => ({
        config: {
            headers: {
                Authorization: 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9sb2NhbGhvc3RcL3dvcmRwcmVzcy1hcGlcL3dvcmRwcmVzczEiLCJpYXQiOjE1OTQzNDU1OTMsIm5iZiI6MTU5NDM0NTU5MywiZXhwIjoxNTk0OTUwMzkzLCJkYXRhIjp7InVzZXIiOnsiaWQiOiIxIn19fQ.K3Tt7WLx0NSUqsa0MA7kwINRIAZuDpGtNpe5WRJRP68'
            }
        },
      dialog: false,
      headers: [
        {
          text: 'Dessert (100g serving)',
          align: 'start',
          sortable: false,
          value: 'title',
        },
        { text: 'id', value: 'id' },
        { text: 'content (g)', value: 'content' },
        { text: 'date (g)', value: 'date' },
        { text: 'status (g)', value: 'status' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      entradas: [],
      editedIndex: -1,
      editedItem: {
        title: '',
        id: '',
        content: '',
        date: '',
        status: '',
      },
      defaultItem: {
        title: '',
        id: '',
        content: '',
        date: '',
        status: '',
      },
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
        limpiar(value){
            return value.replace(/<\/?[^>]+(>|$)/g, "")
        },
      async initialize () {
        try {
                const entradasDB = await this.axios.get('wp/v2/posts');
                await entradasDB.data.forEach(element => {
                    const item = {}
                    item.id = element.id;
                    item.title = element.title.rendered;
                    item.content = this.limpiar(element.content.rendered);
                    item.date = element.date;
                    item.status = element.status;
                    this.entradas.push(item);
                });
            } catch (error) {
                console.log(error);
                
            }
      },

      editItem (item) {
        this.editedIndex = this.entradas.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      async deleteItem (item) {
        const index = this.entradas.indexOf(item)
        const respuesta = confirm('Are you sure you want to delete this item?') && this.entradas.splice(index, 1)
        if(respuesta){
            try {
                await this.axios.delete(`wp/v2/posts/${item.id}`, this.config)
            } catch (error) {
                console.log(error);
            }
        }
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      async save () {
        if (this.editedIndex > -1) {
          
          try {
              const entradaEditada = await this.axios.post(`wp/v2/posts/${this.editedItem.id}`, this.editedItem, this.config);
              Object.assign(this.entradas[this.editedIndex], this.editedItem);
          } catch (error) {
              console.log(error);
          }
        } else {
            const entrada = {
                title: this.editedItem.title,
                content: this.editedItem.content,
                status: 'publish'
            }
            try {
                const entradaDB = await this.axios.post('wp/v2/posts', entrada, this.config);
                this.editedItem.id = entradaDB.data.id;
                this.editedItem.date = entradaDB.data.date;
                this.editedItem.status = entradaDB.data.status;
                this.entradas.push(this.editedItem)

            } catch (error) {
                console.log(error);
            }
            
        }
        this.close()
      },
    },
  }
</script>