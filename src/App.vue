<template>    
  <div id="content">
    <div id="loading">
      Loading...
    </div>

    <div id="postInfo">
      <h1>{{ titre }}</h1>
      <p>Par {{ auteur }}</p>
      <a target="_blank" :href="lien">Lien original</a>
    </div>

    <div id="mainImageDiv">
      <div id="mainImageLeft" @click="changeImage(currentIndex -1)"><div>❰</div></div>
      <img class="hide" id="mainImage" :src="currentUrl" @load="imgLoad">
      <div id="mainImageRight" @click="changeImage(currentIndex +1)"><div>❱</div></div>
    </div>
    
    <div id="scrollBarDiv"> 
      <div id="scrollBar">
        <img class="imageScrollBar" :src="image.data.thumbnail" @click="changeImage(index)" v-for="(image, index) in images" :key="image.data.title" @load="loadDefault(index)">
      </div>
      <div id="imageScrollLeft" @mousedown="scrollHold('left')" @mouseup="stopScrollHold()" @mouseleave="stopScrollHold()"><div>❰</div></div>
      <div id="imageScrollRight" @mousedown="scrollHold('right')" @mouseup="stopScrollHold()" @mouseleave="stopScrollHold()"><div>❱</div></div>
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
      currentUrl: "",
      currentIndex: 1,
      titre: "",
      auteur: "",
      lien: "",
      load: false,
      translateNum: 0,
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

    // Controle toutes les changements d'images
    changeImage(index){
      // verifications pour eviter des erreurs et bugs
      if(index < 0){
        index = 0
      } else if(index > this.images.length){
        index = this.images.length
      }
      if(this.currentIndex == index){
        return 
      }

      // affiche Loading et cache l'image pendant le loading
      document.querySelector("#loading").classList.remove("hide")
      document.querySelector("#mainImage").classList.add("hide")
      
      // enleve le border de l'ancienne image.
      document.querySelectorAll(".imageScrollBar")[this.currentIndex].classList.remove("currentImage") 

      // update l'index
      this.currentIndex = index

      // update l'url
      this.currentUrl = this.images[index].data.url

      // Ajoute le border a l'image courante
      document.querySelectorAll(".imageScrollBar")[index].classList.add("currentImage") 

      // update titre,auteur,lien
      this.titre = this.images[index].data.title
      this.auteur = this.images[index].data.author
      this.lien = this.images[index].data.url

    },

    // Controle le defilement
    scrollHold(side){
      this.interval = setInterval(() => {
        if(side == "right"){
          if(document.querySelector("#scrollBar").clientWidth-window.innerWidth > this.translateNum*-1){
            document.querySelector("#scrollBar").style.transform = `translateX(${this.translateNum -= 5}px)`
          }
        } else {
          if(this.translateNum < 0){
            document.querySelector("#scrollBar").style.transform = `translateX(${this.translateNum += 5}px)`
          }
        }
      }, 10);
    },
    stopScrollHold(){
      clearInterval(this.interval)
    },

    // Affiche la photo et cache Loading... quand la photo est charge
    imgLoad(){
      document.querySelector("#mainImage").classList.remove("hide")
      document.querySelector("#loading").classList.add("hide")
    },

    // Execute la fonction changeImage au debut pour que le site commence avec la premiere image
    loadDefault(i){
      if(i == 0){
        this.changeImage(0)
      }
    }
  },

  mounted() {
    // Exécute les appels à l'API de reddit
    Promise.all([
      this.getImages("wallpaper"),
      this.getImages("wallpapers"),
    ]).then(() => {});

    // Change image avec fleche du clavier
    document.addEventListener("keydown", e => {
      if(e.key == "ArrowRight"){
        this.changeImage(this.currentIndex +1)
      } else if(e.key == "ArrowLeft"){
        this.changeImage(this.currentIndex -1)
      }
    })
  },
}
</script>

<style>
  html, 
  body,
  #app,
  #content {
    height: 100%;
  }

  body {
    background-color: #444;
    margin: 0;
  }

  #postInfo {
    text-align: center;
    color: white;
    font-family: Arial, Helvetica, sans-serif;
  }

  #postInfo h1 {
    font-size: 20px;
    margin: 0;
  }

  #postInfo p {
    margin: 0;
  }

  .imageScrollBar {
    width: 80px;
    height: 80px;
  }

  #scrollBarDiv {
    overflow: hidden;
  }

  #scrollBar {
    display: flex;
    position:fixed;
    bottom: 0px;
    transition: all 1ms linear;
  }

  .imageScrollBar {
    margin: 3px;
    border: 2px solid black;
  }

  #mainImageDiv {
    display: flex;
    justify-content: space-between;
    height: 81%;
    color: white;
  }

  #mainImage {
    margin-top: 10px;
    width: 75%;
    height: 100%;
  }

  .currentImage {
    border: 2px solid red;
  }

  #loading {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: white;
    font-weight: bold;
    font-size: 50px;
  }

  .hide {
    display: none;
  }

  #mainImageLeft,
  #mainImageRight,
  #imageScrollLeft,
  #imageScrollRight {
    display: flex;
    align-items: center;
    font-size: 50px;
    padding: 0px 5px;
    cursor: pointer;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none; 
  }

  #mainImageLeft,
  #mainImageRight {
    background-color: #333;
  }

  #imageScrollLeft,
  #imageScrollRight {
    position: absolute;
    bottom: 0px;
    color: white;
    height: 85px;
    display: flex;
    align-items: center;
    background-color: rgba(3, 3, 3, 0.6);
    margin-bottom: 3px;
  }

  #mainImageLeft {
    padding-right: 10px;
  }

  #mainImageRight {
    padding-left: 10px;
  }

  #imageScrollLeft {
    left: 0px;
    padding-right: 10px;
  }

  #imageScrollRight {
    right: 0px;
    padding-left: 10px;
  }

</style>
