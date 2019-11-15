<script>
	import Image from './Image.svelte';
	import Fullscreen from './Fullscreen.svelte';
	import { onMount, beforeUpdate, afterUpdate} from 'svelte';
  import _ from 'lodash'
  import throttle from 'just-throttle'

  export let tag;

  let index = 0, dto = {}, imageSize = 200 , currentPage = 1, targetSize = 200, isScrollDisabled = false;
  let images=[];
  let windowScrollY, windowInnerWidth = 1000, windowInnerHeight;
  let galleryHeight = 5000, galleryWidth, clientHeight;
  let shouldGetMoreImages, gettingImages = false;
  let fullscreenIdx = -1;
  let hoveredIdx = -1;

  $: imageSize = (galleryWidth / Math.round(windowInnerWidth / targetSize));
  $: galleryHeight = Math.max(document.documentElement.clientHeight, windowInnerHeight || 0);
  $: galleryWidth = windowInnerWidth;
  $: throttledGetImages(tag, 1); //you can also bind functioncalls to changes in their args.
  $: fullscreenDto = images[fullscreenIdx];
  $: galleryLength = images.length;
  $: shouldGetImagesCloseToEnd(fullscreenIdx);


  function shouldGetImagesCloseToEnd(fullscreenIdx) {
    var len = images.length;
    if ((len-fullscreenIdx)<3) {
      throttledGetImages(tag, currentPage);
    }
  }

  function deleteClick(id) {
    var removeIndex = images.findIndex(image => image.id === id);
    images.splice(removeIndex, 1); //mutating objects (and arrays) that are render-involved will not rerender in svelte, see the next line for the solution
    images = images; //this is by design https://github.com/sveltejs/svelte/issues/2362
  }

  function handleFullscreen(id, direction = 0) {
    if (direction === 'close') {
      fullscreenIdx = -1;
      document.body.style.overflow = 'auto';
      return;
    }
    const _index = images.findIndex(image => image.id === id);
    fullscreenIdx = _index + direction;
    document.body.style.overflow = 'hidden';
  }

  function handleHover(index) {
    hoveredIdx = index;
  }

  function handleRotate(index, rotate) {
    images[index].rotate = rotate;
  }

  let throttledGetImages = throttle(getImages, 200, true);
  let throttledHandleScroll = throttle(handleScroll, 50, true);


  function handleScroll() {
    const body = document.body;
    const html = document.documentElement;
    const docHeight = Math.max(body.scrollHeight, body.offsetHeight, html.clientHeight,  html.scrollHeight, html.offsetHeight);
    const windowBottom = windowInnerHeight + windowScrollY;
    shouldGetMoreImages = windowScrollY > 100 && (windowBottom >= docHeight - 500 );
  }

  async function getImages(tag, page = currentPage) {
    if(!gettingImages) {
      gettingImages = true;
      const getImagesUrl = `https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key=522c1f9009ca3609bcbaf08545f067ad&tags=${tag}&tag_mode=any&per_page=100&page=${page}&format=json&nojsoncallback=1`;
      const response = await fetch(getImagesUrl , {
              method: 'GET',
            })
      const res = await response.json();
      if (
        res &&
        res.photos &&
        res.photos.photo &&
        res.photos.photo.length > 0
          )  {
            if(page === 1) {images = []}
            images = (page === 1) ? res.photos.photo : images.concat(res.photos.photo);
          }
      currentPage = page + 1;
      gettingImages = false;
    }
  }
  

  beforeUpdate(() => { //this will happen on scroll because of the bind to scrollY.
    throttledHandleScroll();
  });

  afterUpdate(() => {
    if (shouldGetMoreImages) {
      throttledGetImages(tag, currentPage)};
  });

	function dragstart (ev, index) {
    ev.dataTransfer.setData("index", index);
    ev.dataTransfer.dropEffect = 'move';
  }
  
	function dragover (ev, i) {
    ev.preventDefault();
    hoveredIdx = i;
  }
  
	function drop (ev) {
		ev.preventDefault();
    var dragedIndex = ev.dataTransfer.getData("index");
    var newIndex = hoveredIdx; //why not use dataTransfer? because HTML5 dnd is broken. cant set it on the go so nothing to read at the end.
    const dragImage = images[dragedIndex];
    images.splice(dragedIndex,1);
    images.splice(newIndex,0,dragImage);
    images = images;
	}

</script>


<svelte:window bind:innerWidth={windowInnerWidth} bind:innerHeight={windowInnerHeight} bind:scrollY={windowScrollY}/>
<div class="gallery-root" on:drop={event => drop(event)}
      >

	{#each images as dto, i}
      <span draggable={true} on:dragover={event => dragover(event, i)} on:dragstart={event => dragstart(event, i)}>

    <Image index={i} id={dto.id} rotate={dto.rotate} {handleRotate} large={dto.large} {handleFullscreen} {handleHover} {deleteClick} key={'image-' + dto.id} {dto} {imageSize} {galleryWidth}/>
  </span>
  {/each}
  {#if fullscreenIdx >= 0}
    <Fullscreen {galleryLength} {galleryWidth} {galleryHeight} dto={fullscreenDto} index={fullscreenIdx} {handleFullscreen}/>
  {/if}
</div> 



<style>
.gallery-root {
  text-align: center;
}
</style>