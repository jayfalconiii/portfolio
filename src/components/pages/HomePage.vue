<template>
  <q-layout view="lhr Lpr lfr" class="p-home">
    <div class="l-layout--left" v-if="$q.screen.gt.sm">
      <div class="l-layout--left__l-watermark">
        <q-img
          class="l-watermark__c-icon"
          src="https://drive.google.com/uc?id=1sZYYwv65xzEdhSoLMNsXsl3jlSUZEsm6"
          height="10rem"
          width="10rem"
        />
        <p class="l-watermark__c-name">Jay Falcon</p>
        <Links class="l-watermark__c-links q-mb-lg" />
      </div>
    </div>

    <div
      class="l-layout--right u-box-shadow"
      @wheel.passive="onMouseWheel"
      v-touch-swipe.mouse.right="prevPage"
      v-touch-swipe.mouse.left="nextPage"
    >
      <div class="l-layout--right__c-page">
        <TheBack @click="onPageClick" @gotoFirst="closeBook" />
      </div>
      <div class="l-layout--right__c-page">
        <TheSkills @click="onPageClick" ref="skillPage" />
      </div>
      <div class="l-layout--right__c-page">
        <ThePictures @click="onPageClick" ref="picturesPage" />
      </div>
      <div class="l-layout--right__c-page">
        <TheAchievements @click="onPageClick" ref="achievementsPage" />
      </div>
      <div class="l-layout--right__c-page">
        <TheExperiences @click="onPageClick" ref="experiencePage" />
      </div>
      <div class="l-layout--right__c-page">
        <TheHeader @onClickOpen="nextPage" @gotoContact="gotoBack" />
      </div>
    </div>
  </q-layout>
</template>

<script>
import { defineAsyncComponent } from "vue";
import TheHeader from "@/components/organisms/TheHeader";
import Links from "@/components/atoms/Links";
const TheExperiences = defineAsyncComponent(() =>
  import(
    /* webpackChunkName: "TheExperiences", webpackPreload: true */ "@/components/organisms/TheExperiences"
  )
);
const TheAchievements = defineAsyncComponent(() =>
  import(
    /* webpackChunkName: "TheAchievements", webpackPrefetch: true */ "@/components/organisms/TheAchievements"
  )
);
const TheSkills = defineAsyncComponent(() =>
  import(
    /* webpackChunkName: "TheSkills", webpackPrefetch: true */ "@/components/organisms/TheSkills"
  )
);
const ThePictures = defineAsyncComponent(() =>
  import(
    /* webpackChunkName: "ThePictures", webpackPrefetch: true */ "@/components/organisms/ThePictures"
  )
);
const TheBack = defineAsyncComponent(() =>
  import(
    /* webpackChunkName: "TheBack", webpackPrefetch: true */ "@/components/organisms/TheBack"
  )
);

export default {
  name: "HomePage",
  components: {
    Links,
    TheHeader,
    TheExperiences,
    TheAchievements,
    TheSkills,
    ThePictures,
    TheBack,
  },
  data() {
    return {
      pageProgress: 0,
      pageTimeline: null,
      animIsPaused: false,
      pauseProgress: 0,
    };
  },
  mounted() {
    let animatables = [
      {
        rotateY: 180,
        targets: `.l-layout--right__c-page:nth-of-type(2)`, // TheSkills
      },
      {
        rotateY: 180,
        targets: `.l-layout--right__c-page:nth-of-type(3)`, // ThePictures
      },
      {
        rotateY: 180,
        targets: `.l-layout--right__c-page:nth-of-type(4)`, // TheAchievements
      },
      {
        rotateY: 180,
        targets: `.l-layout--right__c-page:nth-of-type(5)`, // TheExperiences
      },
      {
        rotateY: 180,
        targets: `.l-layout--right__c-page:nth-of-type(6)`, // TheHeader
      },
    ].reverse();

    this.initializeTimeline();
    this.addAnimations(...animatables);
  },
  watch: {
    pageProgress(newValue) {
      if (!this.animIsPaused) {
        const seek = this.pageTimeline.duration * (newValue / 100);
        this.pageTimeline.seek(seek);
      }
    },
  },
  methods: {
    initializeTimeline(option = { autoplay: false }) {
      this.pageTimeline = this.$anime.timeline(option);
    },
    onMouseWheel(e) {
      if (this.$q.platform.is.mobile) {
        return;
      }

      if (e.deltaY < 0 && this.pageProgress >= 0) {
        this.pageProgress -= 5;
      } else if (this.pageProgress < 100) {
        this.pageProgress += 3;
      }
    },
    addAnimations(...options) {
      options.forEach((option) => {
        const offset = option.offset;
        delete option.offset;
        this.pageTimeline.add(
          {
            ...option,
            changeComplete: () => {
              this.disableAllPropagationEvent();
            },
          },
          offset
        );
      });
    },
    disableAllPropagationEvent() {
      this.$refs.experiencePage["$data"].propagateWheelEvent = false;
      this.$refs.picturesPage["$data"].propagateWheelEvent = false;
      this.$refs.skillPage["$data"].propagateWheelEvent = false;
    },
    pauseAnimation() {
      if (this.animIsPaused) return;

      this.animIsPaused = true;
      this.pauseProgress = this.pageProgress; // Store progress
    },
    playAnimation() {
      if (!this.animIsPaused) return;

      this.animIsPaused = false;
      this.pageProgress = this.pauseProgress; // Retrieve saved progress to resume
    },
    onPageClick(e) {
      const isRight = e.clientX >= this.$q.screen.width / 2;
      if (!isRight) {
        this.prevPage();
      } else {
        this.nextPage();
      }
    },
    nextPage() {
      if (this.pageProgress < 100) {
        this.pageProgress += 100 / 5;
      }
    },
    prevPage() {
      if (this.pageProgress > 0) {
        this.pageProgress -= 100 / 5;
      }
    },
    closeBook() {
      this.pageProgress = -25;
    },
    gotoBack() {
      this.pageProgress = 100;
    },
  },
};
</script>

<style lang="scss">
@use "~/src/css/mixins" as *;

.p-home {
  position: fixed;
  display: flex;
}

.l-layout--left {
  position: relative;
  display: flex;
  flex: 1 0;
  justify-content: center;
  height: 100vh;
  background: radial-gradient(
      50% 50% at 50% 50%,
      rgba(0, 0, 0, 0) 0%,
      rgba(0, 0, 0, 0.0446875) 41.15%,
      rgba(0, 0, 0, 0.13) 100%
    ),
    $dark-green;
  margin: 0 0 1rem 0;
  z-index: 1;

  &__l-watermark {
    position: fixed;
    bottom: 0;
  }

  .l-watermark__c-icon {
    background: $warning;
  }

  .l-watermark__c-name {
    margin-top: 0.5rem;
    text-align: center;
    color: white;
    font-size: 1rem;
    font-family: "Praise", cursive;
  }
}

.l-layout--right {
  position: relative;
  height: 100vh;
  width: 100%;
  height: 36px;

  @media (min-width: $breakpoint-md-min) {
    flex: 4 0;
  }

  &__c-page {
    position: absolute;
    height: 100vh;
    width: 100%;
    box-shadow: inset 0px 0px 2px rgba(0, 0, 0, 0.15);

    transform: rotateY(0deg);
    transform-origin: 0% 0%;
    transition: transform 1000ms cubic-bezier(0.645, 0.045, 0.355, 1);
  }
}
</style>
