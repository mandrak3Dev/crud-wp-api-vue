<template>
  <div class="about">
    <h1>This is an crud page</h1>
    <ul>
        <li v-for="item in entradas" :key="item.id">
            {{item.title}} - {{item.content}}
        </li>
    </ul>
  </div>
</template>

<script>
export default {
    data(){
        return {
            entradas: []
        }
    },
    created(){
        this.getEntradas();
    },
    methods: {
        limpiar(value){
            return value.replace(/<\/?[^>]+(>|$)/g, "")
        },
        async getEntradas(){
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
        }
    }
}
</script>