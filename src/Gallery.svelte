<script>
	import Image from './Image.svelte';
	import { onMount, beforeUpdate, afterUpdate} from 'svelte';
  import _ from 'lodash'
  import throttle from 'just-throttle'

  let index = 0, dto = {}, imageSize = 200 , currentPage = 1, targetSize = 200;
  export let tag;
  let images=[];
  let windowScrollY, windowInnerWidth = 1000, windowInnerHeight;
  let galleryHeight = 5000, galleryWidth, clientHeight;
  let div, shouldGetMoreItems, gettingItems = false;
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
  $: newImages = throttledGetImages(tag, 0);

  function delete_Click(id) {
    var tempimages = this.state.images;
    var removeIndex = tempimages.map(function(e) { return e.id; }).indexOf(id);
    tempimages.splice(removeIndex, 1);
    this.setState({images: tempimages});
    
   }

   function large_Click(id) {
    var tempimages = this.state.images;
    var largeIndex = tempimages.map(function(e) { return e.id; }).indexOf(id);
    tempimages[largeIndex].large = !tempimages[largeIndex].large;
    this.setState({images: tempimages});
    
    document.body.style.overflow=(this.isScrollDisabled ? 'auto':'hidden');
    this.isScrollDisabled = !this.isScrollDisabled;
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
  throttle(getImages, 1000, true);


  function handleScroll() {
    const body = document.body;
    const html = document.documentElement;
    const docHeight = Math.max(body.scrollHeight, body.offsetHeight, html.clientHeight,  html.scrollHeight, html.offsetHeight);
    const windowBottom = windowInnerHeight + windowScrollY;
    shouldGetMoreItems = windowScrollY > 100 && (windowBottom >= docHeight - 500 );

  }

  async function getImages(tag, page = currentPage) {
    if(!gettingItems) {
    gettingItems = true;
    console.log('get Images', tag, page);
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
          if(page === 0) { console.log('setting back items');images = []}
          images = (page === 0) ? res.photos.photo : images.concat(res.photos.photo);
        }
      
  currentPage = page + 1;
  gettingItems = false;
  console.log('currentPage is now:', currentPage);
    }
  }
  
  onMount(async () => {
  });

beforeUpdate(() => {
  handleScroll();
});

afterUpdate(() => {
	if (shouldGetMoreItems) {
    throttledGetImages(tag)};
});

</script>



<svelte:window bind:innerWidth={windowInnerWidth} bind:innerHeight={windowInnerHeight} bind:scrollY={windowScrollY}/>
<div bind:this={div} class="gallery-root">

<!-- <div>  images: {images.length}, windowInnerWidth: {windowInnerWidth}, windowInnerHeight: {windowInnerHeight}, windowScrollY: {windowScrollY}, imageSize: {imageSize}, galleryHeight: {galleryHeight}, clientHeight: {document.documentElement.clientHeight}, galleryWidth: {galleryWidth}</div> -->
	{#each images as dto}
	 <Image index={index} id={dto.id} large={dto.large} {large_Click} {delete_Click} key={'image-' + dto.id} {dto} {imageSize} {galleryWidth}/>
	{/each}
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