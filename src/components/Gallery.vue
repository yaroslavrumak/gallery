<template>
  <b-container>
    <b-form inline class="mt-4">
      <label class="sr-only" for="select">Filter primary color</label>
      <b-form-select
        v-model="selected"
        id="select"
        :options="options"
      ></b-form-select>
    </b-form>
    <div class="loader" v-if="loading">
      <img  src="../assets/spiner.gif" alt="loader">
    </div>    
    <div class="image-wrapper" v-else>
      <img 
        class="image-item" 
        v-for="(item, index) in responseData" 
        v-bind:key="index" 
        :src="item.primaryImageSmall" 
        :alt="item.objectName"
      >
    </div>

    <b-modal id="modal-1" @ok="handleOk" ok-only>
        <div class="d-block">Request failed, try again by clicking OK</div>
    </b-modal>
  </b-container>
</template>

<script>
import axios from 'axios'
import rateLimit from 'axios-rate-limit';
import { cacheAdapterEnhancer} from 'axios-extensions';
export default {  
  name: "Gallery",

  data() {
    return {
      selected: null,
      loading: true,
      options: [
        { value: null, text: "Please select an option" },
        { value: "red", text: "Red" },
        { value: "blue", text: "Blue" },
        { value: "green", text: "Green" },
        { value: "none", text: "None" },
      ],
      responseData: []
    };
  },

  mounted(){
    const cachedResponse = window.localStorage.getItem('items');
    cachedResponse && cachedResponse.length ? 
      (this.responseData = [...JSON.parse(cachedResponse)], this.loading = false) : 
      this.getImages()
  },

  methods: {
    showModal() {
        this.$root.$emit('bv::show::modal', 'modal-1')
    },
    handleOk(){
        window.location.reload()
    },
    getImages(){
      const lastThirtyDays = [ ...Array(200).keys() ].map( i => i+200);

      const urls = lastThirtyDays.map(
        (date) =>
          `https://collectionapi.metmuseum.org/public/collection/v1/objects/${date}`
      );

      const http = rateLimit(axios.create({
        adapter: cacheAdapterEnhancer(axios.defaults.adapter)
      }), { maxRequests: 80, perMilliseconds: 1000, maxRPS: 80 })

      function getAllData(urls) {
        return Promise.all(urls.map(fetchData));
      }

      async function fetchData(URL) {
        try {
          const response = await http.get(URL);
          if (response.data && response.status === 200) {
            return response.data;
          }
        } catch (error) {
          console.log('error', error);
        }
      }

      getAllData(urls)
        .then((resp) => {
          const filteredData = resp.filter(i => i !== undefined && i.primaryImageSmall).slice(0, 103)
          this.responseData = [...filteredData]
          const dataToStore = JSON.stringify(filteredData);
          window.localStorage.setItem('items', dataToStore);
          this.loading = false
        })
        .catch(() => {
          this.showModal()
        });
    }
  }
};
</script>
<style lang="scss">
body{
  position: relative;
  form{  
    label{
      padding: 0 15px;  
    }
  }
  .image{
    &-wrapper{
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: space-between;
      flex-wrap: wrap;
    }
    &-item{
      width: 100%;
      max-width: 15%;
      height: 200px;
      padding: 15px;
      object-fit: contain;
    }
  }
  .loader{
    width: 100vw;
    height: 100vh;
    position: absolute;
    display: flex;
    img{
      height: initial;
      width: initial;
      margin: auto;
    }
  }
}

</style>
