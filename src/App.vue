<template>
  <div>
    <div class="container">
      <h2 class="text-center mb-4">MEMORY</h2>
      <div class="config p-3">
        <span>Nombre de tantatives:{{ nbrTentative }}</span>
        <span>Temps de Jeux:{{ timeStamp }}</span>
        <span>Nombre de paires trouvéés:{{ pairesTrouver }}</span>
        <button disabled class="btn btn-danger" id="recommencer" @click="recommencer">
          Recommencer
        </button>
      </div>
      <div class="cards card p-4">
        <div class="fireworks" style="display: none"></div>
        <div
          class="card"
          v-for="(el, i) in pairesImg"
          :key="i"
          :data-id="el.id"
          @click.self="rotate"
        >
          <div class="card_front">
            <div class="card-body">
              <img :src="el.img" alt="" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Fireworks } from "fireworks-js";
export default {
  name: "App",
  data() {
    return {
      images: [
        require("./assets/img/Maki.jpg"),
        require("./assets/img/pigeon.jpg"),
        require("./assets/img/pinguin.png"),
        require("./assets/img/cheval.jpg"),
      ],
      pairesImg: [],
      nbrTentative: 0,
      pairs: [],
      pairesTrouver: 0,
      timeStamp: 0,
      timer: null,
      startTimer: false,
      audio: new Audio(require("../son/videoplayback.webm")),
    };
  },
  created() {
    this.melange();
  },
  methods: {
    melange() {
      let createPairs = [];
      for (let i = 0; i < this.images.length; i++) {
        createPairs.push(
          {
            id: i,
            img: this.images[i],
          },
          {
            id: i,
            img: this.images[i],
          }
        );
      }
      this.pairesImg = createPairs.sort(() => 0.5 - Math.random());
    },
    rotate(e) {
      this.startTimer = true;
      document.querySelector("#recommencer").removeAttribute("disabled");
      if (this.pairs.length < 2) {
        e.target.classList.toggle("rotate");
        this.pairs.push(e.target);
        if (this.pairs.length === 2) {
          this.nbrTentative++;
          if (this.pairs[0].dataset.id === this.pairs[1].dataset.id) {
            this.pairesTrouver++;
            this.pairs = [];
          } else {
            setTimeout(() => {
              this.pairs[0].classList.remove("rotate");
              this.pairs[1].classList.remove("rotate");
              this.pairs = [];
            }, 1000);
          }
        }
      }
      if (this.pairesTrouver === 4) {
        clearInterval(this.timer);
        document.querySelector("#recommencer").innerHTML = "Rejouer";
        this.melange();
        this.reset();
        this.gagner();
        // this.audio.pause();
        // this.audio.currentTime = 0;
      }
    },
    recommencer(e) {
      this.nbrTentative = 0;
      this.pairesTrouver = 0;
      this.timeStamp = 0;
      e.target.setAttribute("disabled", "");
      clearInterval(this.timer);
      this.reset();
      this.melange();
      document.querySelector(".fireworks").style.display = "none";
      this.audio.pause();
      this.audio.currentTime = 0;
    },

    reset() {
      Array.from(document.querySelectorAll(".card")).forEach((card) => {
        card.classList.remove("rotate");
      });
    },

    gagner() {
      const container = document.querySelector(".fireworks");
      document.querySelector(".fireworks").style.display = "block";
      const fireworks = new Fireworks(container, {
        acceleration: 1.2,
        particles: 100,
        trace: 6,
      });

      this.audio.play();
      fireworks.start();
    },
  },

  watch: {
    startTimer() {
      if (this.startTimer) {
        let min = 0;
        let sec = 0;
        this.timer = setInterval(() => {
          sec++;
          if (sec < 10) {
            sec = "0" + sec;
          }
          if (sec >= 60) {
            sec = 0;
            min++;
          }

          this.timeStamp = min + " : " + sec;
        }, 1000);
      }
    },
  },
};
</script>

<style lang="scss" src="../src/assets/app.scss"></style>
