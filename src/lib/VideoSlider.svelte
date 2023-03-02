<svelte:window 
	on:touchmove={move} 
	on:touchend={end} 
	on:mousemove={move} 
	on:mouseup={end} 
	on:resize={resize}    
/>

<script>
    let seekable
    let paused_one = true
    let paused_two = true

	let hideOnSlide = true,
        imgOffset = null,
		sliding = false,
		contain = false,
		overlay = true,
		offset = 0.5,
		before = '',
		after = '',
		img;
    
    let volume_one = 0.5
    let volume_two = 0.5
        
	function resize(e) {        
		imgOffset = (e.type === 'load' ? e.target : img).getBoundingClientRect();
	}
	function move(e) {
		if (sliding && imgOffset) {
			let x = (e.touches ? e.touches[0].pageX : e.pageX) - imgOffset.left;
			x = x < 0 ? 0 : ((x > w) ? w : x);
			offset = x / w;            
            volume_one = 1-offset
            volume_two = offset            
		}
	}
	function start(e) {
		sliding = true;
		move(e);
        paused_one=false;
        paused_two=false;
	}
	function end() {
		sliding = false;
	}


	$: w = imgOffset && imgOffset.width;
	$: h = imgOffset && imgOffset.height;
	$: x = w * offset;
	$: opacity = hideOnSlide && sliding ? 0 : 1;
	$: style = contain ? `width:100%;height:500px;` : `width:${w}px;height:${h}px;`;
    $: {        
        if (seekable && seekable.length>0){
            imgOffset = img.getBoundingClientRect();
        }
    }
    
    let waiting = 0
    
    const notifyLoaded = () => {
        console.log('loaded!')
    }
    
    const onload = el => {
        waiting++
        el.addEventListener('load', () => {
            waiting--
            if (waiting === 0) {
                notifyLoaded()
            }
        })
    }


  
	export { before, after, offset, overlay, contain, hideOnSlide };
</script>

<div class="container" {style} on:touchstart={start} on:mousedown={start}>
	<video use:onload
		bind:this={img}
        bind:paused={paused_one}
        bind:seekable={seekable}
		src={after} 
        bind:volume={volume_one}
		alt="after" 
		on:mousedown|preventDefault 
		on:load={resize} 
		{style}
	/>
	<video 
		src={before} 
		alt="before" 
        bind:paused={paused_two}
        bind:volume={volume_two}
		on:mousedown|preventDefault 
		style="{style}clip:rect(0, {x}px, {h}px, 0);"
	/>
	{#if overlay}
	<div class="overlay" style="opacity:{opacity}"></div>
	{/if}
	<div class="before-label" style="opacity:{opacity}">
		<slot name="before"></slot>
	</div>
	<div class="after-label" style="opacity:{opacity}">
		<slot name="after"></slot>
	</div>
	<div class="handle" style="left: calc({offset * 100}% - 20px)">
		<div class="arrow-left"></div>
		<div class="arrow-right"></div>
	</div>
</div>

<style>
	.container {
		overflow: hidden;
		position: relative;
		box-sizing: content-box;  
        height:300px;
	}
	.container video {
		top: 0;
		left: 0;
		z-index: 20;
		display: block;
		max-width: 100%;
		user-select: none;
		object-fit: cover;
		position: absolute;
        
	}
	.overlay {
		top: 0;
		opacity: 0;
		z-index: 25;
		width: 100%;
		height:100%;
        
		position: absolute;
		transition: opacity .5s;
		background: rgba(0, 0, 0, .5);
	}
	.before-label, .after-label {
		top: 0;
		bottom: 0;
		z-index: 25;
		user-select: none;
		position: absolute;
	}
	.before-label { left: 0; }
	.after-label { right: 0; }
	.container:hover > .overlay { opacity: 1; }
	.handle {
		z-index: 30;
		width: 40px; 
		height: 40px;
		cursor: move;
		background: none;
		margin-top: -4px;
		margin-left: -4px;
		user-select: none;
		position: absolute;
		border-radius: 50px;
		top: calc(50% - 20px);
		border: 4px solid white;
	}
	.handle:before, .handle:after {
		content: "";
		height: 9999px;
		position: absolute;
		left: calc(50% - 2px);
		border: 2px solid white;
	}
	.handle:before { top: 40px; }
	.handle:after { bottom: 40px; }
	.arrow-right, .arrow-left {
		width: 0;
		height: 0;
		user-select: none;
		position: relative;
		border-top: 10px solid transparent;
		border-bottom: 10px solid transparent;
	}
	.arrow-right {
		left: 23px;
		bottom: 10px;
		border-left: 10px solid white;
	}
	.arrow-left {
		left: 7px;
		top: 10px;
		border-right: 10px solid white;
	}
</style>
