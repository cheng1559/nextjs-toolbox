<script>
import axios from 'axios'
export default {
  data() {
    return {
      musicList: [
        'Feliz Navidad.mp3',
        'Hallelujah.mp3',
        'Last Christmas.mp3',
        'O Christmas Tree.mp3',
        'All I Want For Christmas Is You.mp3'
      ],
      currentMusicIndex: 0,
      backgroundMusic: null,
      canvas: "",
      ctx: "",
      W: "",
      H: "",
      angle: -10,
      mp: 500,
      particles: [],
      t: 0,
      isInfo: false,
      abotuImage: 'http://cdn.lixsx.net/lixsx.jpg',
      bgImage: 'http://cdn.lixsx.net/main-bg.jpg',
      imageSrc: "",
      state: "please upload your profile"
    };
    
  },
  beforeMount() {
    // window.addEventListener('scroll', this.disableScroll);
    // window.addEventListener('touchmove', function (e) {
    //   e.preventDefault();
    // }, { passive: false });
  },
  beforeDestroy() {
    // window.removeEventListener('scroll', this.disableScroll);
  },
  mounted() {
    this._initCavas();
  },
  computed: {
    currentMusic() {
      return this.musicList[this.currentMusicIndex];
    }
  },
  methods: {
    get_return() {
      console.log('send')
      axios.get('https://hatter-hat-vhcmaxttpc.cn-hangzhou.fcapp.run?' + this.imageSrc)
      .then(response => {
        console.log(response.data);
        this.imageSrc = 'data:image/png;base64,' + response.data;
        this.state = 'finish';
      });
    },

    compressImage(file, options) {
      return new Promise((resolve) => {
        const reader = new FileReader();

        reader.onload = (e) => {
          const img = new Image();

          img.onload = () => {
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');

            const maxWidth = options.maxWidth || img.width;
            const maxHeight = options.maxHeight || img.height;
            const scale = Math.min(maxWidth / img.width, maxHeight / img.height);

            canvas.width = img.width * scale;
            canvas.height = img.height * scale;

            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

            const compressedDataURL = canvas.toDataURL('image/jpeg', 0.8);
            resolve(compressedDataURL);
          };

          img.src = e.target.result;
        };

        reader.readAsDataURL(file);
      });
    },

    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        const f = URL.createObjectURL(file);
        this.compressImage(file, { maxWidth: 400, maxHeight: 400 })
        .then((compressedImage) => {
          this.imageSrc = compressedImage;
        })
      }
    },

    playMusic() {
      this.get_return();
      this.state = 'loading...';
      const popSound = new Audio('pop.wav');
      popSound.play();
      
      this.currentMusicIndex = Math.floor(Math.random() * this.musicList.length);
      if (this.backgroundMusic && !this.backgroundMusic.paused) {
        this.backgroundMusic.pause();
      }
      this.backgroundMusic = new Audio(this.currentMusic);
      this.backgroundMusic.play();
    },
    disableScroll() {
      // window.scrollTo(0, 0); // 将滚动位置重置为顶部
    },
    showInfo() {
      this.isInfo = true;
    },
    _initCavas() {
      this.canvas = document.getElementById("canvas");
      this.ctx = this.canvas.getContext("2d");

      //canvas dimensions
      this.W = window.innerWidth - 30;
      this.H = window.innerHeight - 10;
      this.canvas.width = this.W;
      this.canvas.height = this.H;

      //snowflake particles
      this.particles = [];
      for (var i = 0; i < this.mp; i++) {
          this.particles.push({
              x: Math.random() * this.W, //x-coordinate
              y: Math.random() * this.H, //y-coordinate
              r: Math.random() * 3 + 1, //radius
              d: Math.random() * this.mp //density
          })
      }
      clearInterval(localStorage.getItem('interval'));
      localStorage.setItem('interval', setInterval(this.draw, 15));
  },
  draw() {
      this.ctx.clearRect(0, 0, this.W, this.H);

      this.ctx.fillStyle = "rgba(255, 255, 255, 0.8)";
      this.ctx.beginPath();
      for (var i = 0; i < this.mp; i++) {
          var p = this.particles[i];
          this.ctx.moveTo(p.x, p.y);
          this.ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2, true);
      }
      this.ctx.fill();
      this.update();
  },
  update() {
      // this.angle += 0.01;
      for (var i = 0; i < this.mp; i++) {
          var p = this.particles[i];
          p.y += Math.cos(this.angle + p.d) + 1 + p.r / 2;
          p.x += Math.sin(this.angle) * 2;

          if (p.x > this.W || p.x < 0 || p.y > this.H) {
              if (i % 3 > 0) {
                  this.particles[i] = {
                      x: Math.random() * this.W,
                      y: -10,
                      r: p.r,
                      d: p.d
                  };
              } else {
                  if (Math.sin(this.angle) > 0) {
                      //Enter fromth
                      this.particles[i] = {
                          x: -5,
                          y: Math.random() * this.H,
                          r: p.r,
                          d: p.d
                      };
                  } else {
                      this.particles[i] = {
                          x: this.W + 5,
                          y: Math.random() * this.H,
                          r: p.r,
                          d: p.d
                      };
                  }
              }
          }
      }
    }
  }
};

</script>

<template>
  <div class="background">
    <h1 class="title">Santa Hatter</h1>
    
    <div class="box">
      <input type="file" @change="handleFileChange" />
    </div>
    <div class="box">
      <img :src="imageSrc" alt="Decoded Image" class="img">
    </div>

    <h1 class="label">{{ state }}</h1>

    <div class="empty-space"></div>
    <div class="box">
      <button class="button" @click="playMusic">
        Click Me
      </button>
    </div>
    

    <h1 class="music">Current Music: {{ currentMusic }}</h1>
  </div>
  <canvas id="canvas" ref="canvas" class="snow"></canvas>
  
</template>

<style>

:root {
  --test: 20vw;
}

.title {
  text-align: center;
  margin-top: 20vw;
  font-size: 15vw;
  font-family: Caslon;
  color: white;
}

.button {
  /* text-align: center; */
  font-size: 6vw;
  font-family: Pixeloid Mono;
  color: white;

  background-image: url('/button_normal.png');
  background-color: rgba(0, 0, 0, 0);
  background-size: cover;

  width: 40vw;
  height: 16vw;

  /* padding: 10px 20px; */
  border: none;
  cursor: pointer;
  /* 正常状态背景 */
}

/* 悬停状态 */
.button:hover {
  background-image: url('/button_hover.png');
}

/* 点击状态 */
.button:active {
  background-image: url('/button_pressed.png');
}

.box {
  display: flex;
  justify-content: center;
  align-items: center;
  height: auto;
}

.background {
  position: absolute;
  left: 0;
  top: 0;
  background-image: url('/background.webp');
  background-size: cover;
  background-position: center;
  height: 100vh;
  width: 100vw;
}

.empty-space {
  margin-bottom: 5vh;
}

.center-container {
  /* display: flex; */
  justify-content: center;
  height: 100vw;
}

.music {
  text-align: center;
  position: fixed;
  bottom: 0;
}

.snow {
  /* z-index: 100;r */
  position: fixed;
  top: 0;
  left: -100vw;
  height: 200vh;
  width: 200vw;
  pointer-events: none;
}

.img {
  display: flex;
  justify-content: center;
  align-items: center;
  left: 50vw;
  height: 30vh;
  width: auto;
}

.label {
  text-align: center;
  color: black;
  font-family: Caslon;
  font-size: 30px;
}

</style>