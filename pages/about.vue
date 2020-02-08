<template>
  <div id="about-page" class="page-wrapper about-page content-page">
    <site-hero
      title="who am I"
      subtitle="is writing a bio page narcissism?"
      image="/uploads/about-hero.jpg"
    ></site-hero>
    <main-section theme="sidebar-right">
      <template v-slot:default>
        <div class="content">
          <h3>Professional Journey</h3>
            
            <h4>Data Scientist <small> - Current Role since March 2018 </small> </h4>
              <p>
              Analytics and Artificial Intelligence, Cummins Inc., Columbus, IN, US
              </p>
              <p>
              Current responsibilities/initiatives include, 
                <ul>
                  <li>Design, implementation, and maintenance of Data Science algorithms’ execution pipeline for telematics data.</li>
                  <li>Implementing CI/CD devops workflows and designing testing Strategy. </li>
                  <li>Research, and implementation of Semi/Unsupervised/Supervised machine learning algorithms for detecting statistical anomalies in the engine telematics data (time series data).</li>
                  <li>Current and ongoing research: adaptive kernel principal component analysis, time series clustering, engine fail code isolation techniques, spark streaming with Custom aggregations.</li>
                </ul>          
              </p>
              <p>
              Tool stack: Python, Pyspark, Scala, R, SparkR, Hadoop, SQL, git, Jenkins, Azure Databricks, Azure HDInsights, Azure DevOps, PowerBI, Rstudio, Jupyter.
              </P>
              
              <h4>Machine Data Analytics Intern <small> - May 2017 to August 2017; 3 months </small> </h4>
              <p>
              Machine Data Analytics, Cummins Inc., Columbus, IN, US
              </p>
              <p>
              Succeeded in the machine data analytics team to expand expert diagnostics system/fault code analyzer capability. My project concluded with 
                <ul>
                  <li>Creating a visualization platform for live onboard diagnostics data.</li>
                  <li>Finding candidate diagnosis for implementing prediction algorithms.</li>
                  <li>Attempting the prediction of engine disability events using statistical process control techniques and advanced machine learning algorithms.</li>
                </ul>          
              </p>
              <p>
              Tool stack: Python, R, SQL, git, Azure HDInsights, PowerBI, Rstudio, Jupyter.
              </P>
              
              <h4>Applied Mechanics Engineer <small> - Sept. 2014 to June 2016; 21 months </small> </h4>
              <p>
              Applied Mechanics, Cummins India Limited, Pune, India
              </p>
              <p>
              I performed various kinds of structural analyses on different diesel engine components, sub-assemblies, and assemblies 
              by simulating structural loads generated during different duty cycles. Achieved excellent ratings consistently 
              through result-oriented work aimed for the excellence in,
                <ul>
                  <li>Producing accurate finite elemental analyses results.</li>
                  <li>Effective report presentations.</li>
                  <li>Global client engagement (US, India, UK, China & Russia).</li>
                  <li>Various initiatives in employee engagement activities.</li>
                </ul>          
              </p>
              <p>
              Tool stack: <span>&#8226;</span> Finite elemental Software: Ansys, Fe-Safe, Ansys customization toolkit(python) 
              <span>&#8226;</span> Design Software: Ansys DM, ProEngineer <span>&#8226;</span> Statistical software: Minitab
              <span>&#8226;</span> Everyday work software: MS office suite
              </P>
              
              <h4>Engineer Intern  <small> - Nov. 2013 to April 2014; 6 months </small> </h4>
              <p>
              Quality Assurance, Ceat Tyres Limited, Mumbai, India.
              </p>
              <p>
              Accomplished standardization of CEAT management system’s implementation throughout the functionality of company 
              through teamwork. It reflected into company’s performance for getting standardized and laying foundation for 
              Deming’s price in quality. 
              </p>
              <p>
              My work concluded with,
                <ul>
                  <li>creating handbooks/manuals for the Original equipment manufacturing sales department, finance department, and supply chain management department which included process flow maps, SOPs, and Rules and responsibilities mapping.</li>
                  <li>Everyday tasks included meeting senior executives, understanding different process flows, identifying/mapping them into distinct steps, preparing flow charts, drafting standard operating procedures at the operations level, identifying gaps in the flows, and defining key performance indicators.</li>
                  <li>And my undergraduate project: Engineering redesign of inspection bay for tyres with manufacturing defects.</li>
                </ul>  
              </P>
              <p>
              Tool stack: <span>&#8226;</span> time and motion study, shopfloor management <span>&#8226;</span> Design Software: Solidworks 3D, NX5 <span>&#8226;</span> Everyday work software: MS office suite
              </P>
          
          <h3>Education</h3>
          
          <quote-card />
          
          <h3>Resume</h3>

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
