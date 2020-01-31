<template>
  <div id="about-page" class="page-wrapper about-page content-page">
    <site-hero
      title="Who am I"
      subtitle="is writing a bio page narcissism?"
      image="/uploads/about-hero.jpg"
    ></site-hero>
    <main-section theme="sidebar-right">
      <template v-slot:default>
        <div class="content">
          <h3>Where I come from</h3>
          <p>
          under construction
          </p>
          
          <quote-card />
          
          <h3>Resume</h3>
          <figure>
                <opti-image
                  :src="require('~/assets/uploads/Resume_Ashish_2020-1.jpg').src"
                  :srcset="require('~/assets/uploads/Resume_Ashish_2020-1.jpg').srcSet"
                />
              </figure>
              <br />
        </div>

        <div class="tile is-ancestor">
          <div class="tile is-parent">
            <article class="tile is-child box">
              <p class="title">
                Team Pravega
              </p>
              <p class="subtitle">
                Student prototype of Formual one car
              </p>
              <figure class="image is-1by1">
                <opti-image 
                :src="require('~/assets/uploads/pravega.jpg').src"
                :srcset="require('~/assets/uploads/pravega.jpg').srcSet" width="500" height="500" />
              </figure>
            </article>
          </div>
          <div class="tile is-parent">
            <article class="tile is-child box">
              <p class="title">
                Team Abadha
              </p>
              <p class="subtitle">
                All terrain vehicle
              </p>
              <figure class="image is-1by1">
                <opti-image 
                :src="require('~/assets/uploads/abadha.jpg').src"
                :srcset="require('~/assets/uploads/abadha.jpg').srcSet" width="500" height="500" />
              </figure>
            </article>
          </div>
          <div class="tile is-parent">
            <article class="tile is-child box">
              <p class="title">
                And I bake
              </p>
              <p class="subtitle">
                sourdough bread
              </p>
              <figure class="image is-1by1">
                <opti-image 
                :src="require('~/assets/uploads/sourdough.jpg').src"
                :srcset="require('~/assets/uploads/sourdough.jpg').srcSet" width="500" height="500" />
              </figure>
            </article>
          </div>
        </div>
      </template>

      <template v-slot:sidebar>
        <h3 class="subtitle is-4">
          Latest Posts
        </h3>
        <posts-grid :per-row="1" :number="2" />
      </template>
    </main-section>
  </div>
</template>
<script>
import QuoteCard from '~/components/cards/QuoteCard'
export default {
  head() {
    return {
      title: `About | ${this.$siteConfig.siteName}`
    }
  },
  components: { QuoteCard }
}
</script>
