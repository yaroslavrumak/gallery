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
    <div class="image-wrapper">
      <img class="image-item" v-for="(item, index) in responseData" v-bind:key="index" :src="item.primaryImageSmall" :alt="item.objectName">
    </div>
  </b-container>
</template>

<script>
import axios from 'axios'
import rateLimit from 'axios-rate-limit';
import analyze from 'rgbaster'
export default {  
  name: "HelloWorld",

  data() {
    return {
      selected: null,
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
    this.getImages()

    const container = document.querySelector('.image-container');

    const fac = new FastAverageColor();
    fac.getColorAsync(document.querySelectorAll('.image-item'))
    .then(color => {
      console.log("color", color);
        container.style.backgroundColor = color.rgba;
        container.style.color = color.isDark ? '#fff' : '#000';
    })
    .catch(e => {
        console.log(e);
    });
  },

  сomputed: {
   
  },

  methods: {
    getImages(){
      const lastThirtyDays = [ ...Array(200).keys() ].map( i => i+200);

      const urls = lastThirtyDays.map(
        (date) =>
          `https://collectionapi.metmuseum.org/public/collection/v1/objects/${date}`
      );

      const http = rateLimit(axios.create(), { maxRequests: 50, perMilliseconds: 1000, maxRPS: 50 })

      function getAllData(urls) {
        return Promise.all(urls.map(fetchData));
      }

      async function fetchData(URL) {
        try {
          const response = await http.get(URL);
          // console.log('response', response);
          if (response.data && response.status === 200) {
            console.log('test', response);
            return response.data;
          }
        } catch (error) {
          console.log('error2', error);
        }
      }

      getAllData(urls)
        .then((resp) => {
          const filteredData = resp.filter(i => i !== undefined && i.primaryImageSmall).slice(0, 103)
          // filteredData.map(i => {
          //   const result = await analyze(i.primaryImageSmall, { ignore: [ 'rgb(255,255,255)', 'rgb(0,0,0)' ] })
          // })
          // const result = await analyze('/image.png', { ignore: [ 'rgb(255,255,255)', 'rgb(0,0,0)' ] })
          this.responseData = [...filteredData]
        })
        .catch((e) => {
          console.log(e.message);
        });
    }
  }
};
</script>
<style lang="scss">
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
    max-width: 33%;
    height: 500px;
    padding: 15px;
  }
}
</style>
