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
            I did my schooling and graduated junior college in Jalgaon, India. 
            Moved to Mumbai in 2010 to attend University of Mumbai (Fr. Conceicao Rodrigues College of Engineering, Bandra, Mumbai) 
            in pursuit of Bachelor's in Engineering (Production Engineering).
          </p>
          <p>
            After graduating in 2014 (first class with distinction), I joined Cummins India Ltd. as Applied Mechanics Engineer. 
            Couple of years in structural analysis, and I was off for Master os Science in Industrial Engineering at University of 
            Illinois at Chicago.  
          </p>
          
          <p>
            During Masters' I become passionate about Data Science. I joined Cummins Inc. as a Data Scientist in March, 2018 and 
            since then the journey has been full of excitement.   
          </p>
          
          <quote-card />
          
          <h3>Resume</h3>
          <p>
          <img src="/uploads/Resume_Ashish_2020-1.jpg" alt="resume image">
          </p>
        </div>

        <div class="tile is-ancestor">
          <div class="tile is-parent">
            <article class="tile is-child box">
              <p class="title">
                Cool Stuff We Do
              </p>
              <p class="subtitle">
                Sooo Cool
              </p>
              <figure class="image is-1by1">
                <opti-image width="500" height="500" />
              </figure>
            </article>
          </div>
          <div class="tile is-parent">
            <article class="tile is-child box">
              <p class="title">
                Other Cool Things
              </p>
              <p class="subtitle">
                Just as cool
              </p>
              <figure class="image is-1by1">
                <opti-image width="500" height="500" />
              </figure>
            </article>
          </div>
          <div class="tile is-parent">
            <article class="tile is-child box">
              <p class="title">
                All the cool
              </p>
              <p class="subtitle">
                that is all
              </p>
              <figure class="image is-1by1">
                <opti-image width="500" height="500" />
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
