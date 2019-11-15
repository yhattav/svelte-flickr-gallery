<script>
	import Image from './Image.svelte';
	import Fullscreen from './Fullscreen.svelte';
	import { onMount, beforeUpdate, afterUpdate} from 'svelte';
  import _ from 'lodash'
  import throttle from 'just-throttle'

  let index = 0, dto = {}, imageSize = 200 , currentPage = 1, targetSize = 200, isScrollDisabled = false;
  export let tag;
  let images=[];
  let windowScrollY, windowInnerWidth = 1000, windowInnerHeight;
  let galleryHeight = 5000, galleryWidth, clientHeight;
  let shouldGetMoreImages, gettingImages = false;
  let fullscreenIdx = -1;
  let hoveredIdx = -1;
//   function moveImage(dragIndex, hoverIndex) {
// 		const { images } = this.state
//     const dragImage = images[dragIndex]
//     images.splice(dragIndex,1);
//     images.splice(hoverIndex,0,dragImage);
// 		this.setState({
// 				images
// 			});
		
// 	}

  $: imageSize = (galleryWidth / Math.round(windowInnerWidth / targetSize));
  $: galleryHeight = Math.max(document.documentElement.clientHeight, windowInnerHeight || 0);
  $: galleryWidth = windowInnerWidth;
  $: throttledGetImages(tag, 0); //you can also bind functioncalls to changes in their args.
  $: fullscreenDto = images[fullscreenIdx];
  $: galleryLength = images.length;

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
      console.log('handle hover' , index);
      hoveredIdx = index;
  }

  function handleRotate(index, rotate) {
    images[index].rotate = rotate;
    console.log(images[index].rotate);
  }

function arrow_Click(id,direction) {
  var tempimages = this.state.images;
  var largeIndex = tempimages.map(function(e) { return e.id; }).indexOf(id);
  var newIndex = largeIndex+direction;
  tempimages[largeIndex].large = !tempimages[largeIndex].large;
  tempimages[newIndex].large = !tempimages[newIndex].large;
  this.getCloseToEnd(newIndex,tempimages.length)
  this.setState({images: tempimages});

}

// function getCloseToEnd(index,len) {
//    var len = this.state.images.length;
//    if ((len-index)<3) {
//      this.getImages(tag,currentPage);
//     }
// }
let throttledGetImages =
  throttle(getImages, 200, true);


  function handleScroll() { //TODO should throttle scroll
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
          if(page === 0) {images = []}
          images = (page === 0) ? res.photos.photo : images.concat(res.photos.photo);
        }
      
  currentPage = page + 1;
  gettingImages = false;
    }
  }
  

beforeUpdate(() => {
  handleScroll();
    console.log('before update - try to lower me?')

});

afterUpdate(() => {
	if (shouldGetMoreImages) {
    throttledGetImages(tag)};
});

	export function dragstart (ev, index) {
    console.log('dragstart', index)
    ev.dataTransfer.setData("index", index);
    ev.dataTransfer.dropEffect = 'move';

  }
  
	export function dragover (ev, i) {
    ev.preventDefault();
    hoveredIdx = i;
    console.log('draged over', hoveredIdx);
  }
  
	export function drop (ev) {
        console.log('drop')

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

<!-- <div>  images: {images.length}, windowInnerWidth: {windowInnerWidth}, windowInnerHeight: {windowInnerHeight}, windowScrollY: {windowScrollY}, imageSize: {imageSize}, galleryHeight: {galleryHeight}, clientHeight: {document.documentElement.clientHeight}, galleryWidth: {galleryWidth}</div> -->
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

.gallery-header {
  background-color: #222;
  padding: 10px;
  color: white;
  font-size: 20pt;
}

.gallery-intro {
  font-size: 15pt;
}
</style>