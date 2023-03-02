Move da slider <br/>
<script >
    import "../app.css"
    import VideoSlider from "$lib/VideoSlider.svelte"
    import { fade } from 'svelte/transition';
    import { tweened } from 'svelte/motion';
    
    //fade in out the initial text
    const MIN_OPACITY = 0.2;
    const MAX_OPACITY = 0.8;
    const opacity = tweened(MIN_OPACITY);
    let hidden = false

    function animate(){
      opacity.set(MAX_OPACITY, { duration: 1000 })
      .then(() => opacity.set(MIN_OPACITY, { duration: 1000 }))
      .then(()=>{animate()})
    }
     
    const hide = () => {
      hidden=true
      console.log("Aaa")
    }
    animate()
</script>

{#if !hidden}
  <div class="intro" on:click={hide} in:fade out:fade>
    <img src="/logo-header.svg" alt="tap to play"/>
    <div  style="opacity: {$opacity}" >Press to continue</div>
  </div>
{/if}

<VideoSlider  before="/MALE1.mp4" after="/MALE2.mp4" contain={true}>
	<span slot="before">BEFORE</span>
	<span slot="after">AFTER</span>
</VideoSlider>

<style>
  .intro{
    position:absolute;
    color:#666;
    flex-direction: column;
    z-index: 100;
    height:100vh;
    background-color:#000;
    width:100vw;
    display:flex;
    justify-content:space-around;
    align-items: center;
    text-transform: uppercase;
    font-family: sans-serif;
    cursor:pointer
  }
</style>