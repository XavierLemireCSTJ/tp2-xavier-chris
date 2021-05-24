<template>
  <div>
    <div v-if="images.length > 0">
      {{images.length}}
      <img :src="images[1].data.url">
    </div>
  </div>
</template>

<script>
import shuffle from './shuffle'

export default {
  name: 'App',
  components: {},
  data() {
    return {
      images: [],
    }
  },
  methods: {
    // Fetch les 30 plus récents 'posts' du subreddit passé en paramètre
    getImages(name){
      let url = "https://www.reddit.com/r/" + name + ".json?limit=30"

      fetch(url).then(resp =>{
        return resp.json()
      }).then(data =>{
        // Récupère les 30 posts, puis tri et ajoute seulement les 'posts' avec des images (extensions: jpg/jpeg/png/gif/webp) à l'array post_image
        let posts = data.data.children
        let post_image = [] 

        posts.forEach(post => {
          let post_url = post.data.url
          if(post_url.endsWith(".jpg") || post_url.endsWith(".jpeg") || post_url.endsWith(".png") || post_url.endsWith(".gif") || post_url.endsWith(".webp")){
            
            post_image.push(post)
          }
        })

        // Retourne les posts dans un ordre aléatoire
        shuffle(post_image)

        // Ajoute les 'posts' d'image à la propriété images
        this.images.push.apply(this.images, post_image)
      })
    },
  },
  mounted() {

    // Exécute les appels à l'API de reddit
    Promise.all([
      this.getImages("wallpaper"),
      this.getImages("wallpapers"),
    ]).then(() => {});
  }
}
</script>

<style>

</style>
