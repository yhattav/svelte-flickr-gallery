<script>
  import Icon from 'fa-svelte'
  import { faTimes } from '@fortawesome/free-solid-svg-icons/faTimes'
  import { faChevronLeft } from '@fortawesome/free-solid-svg-icons/faChevronLeft'
  import { faChevronRight } from '@fortawesome/free-solid-svg-icons/faChevronRight'
  let sizes=[{width: 100, height: 100, source:''}];
	export let galleryWidth, galleryHeight, dto = {}, index, galleryLength, handleFullscreen;
  $: imageWidth = sizes[sizes.length-1] ? sizes[sizes.length-1].width : sizes[sizes.length].width;
  $: imageHeight = sizes[sizes.length-1] ? sizes[sizes.length-1].height : sizes[sizes.length].height;
  $: imageRatio = imageHeight / imageWidth;
  $: galleryRatio = galleryHeight / galleryWidth;
  $: isVertical = (imageRatio >= galleryRatio);
  $: slideWidth = isVertical ? galleryHeight / imageRatio * 0.9 : galleryWidth * 0.9;
  $: slideHeight = isVertical ? galleryHeight * 0.9 : galleryWidth*imageRatio * 0.9;
  $: getSizes(dto);
  $: fullscreenUrl = urlFromSizes(sizes);
  $: nextArrowTitle = `Next image: ${index+1} / ${galleryLength}`;
  $: backArrowTitle = `Previous image: ${index-1} / ${galleryLength}`;
  $: slideTop =  (galleryHeight-slideHeight) / 2;
  $: slideLeft = (galleryWidth-slideWidth) / 2;

  function urlFromSizes(sizes) {
    return getBestFittingRes(sizes).source;
  }

  function getBestFittingRes(sizes) {
    let bestSize = sizes.find(size => (size.height >= galleryHeight || size.width >= galleryWidth));
    return bestSize || sizes[sizes.length-1];
  }

  function handleIconClick(direction) {
    handleFullscreen(dto.id, direction)
  }

  async function getSizes(dto) {
    const getSizesUrl = `https://api.flickr.com/services/rest/?method=flickr.photos.getSizes&api_key=522c1f9009ca3609bcbaf08545f067ad&photo_id=${dto.id}&format=json&nojsoncallback=1`;
    const response = await fetch(getSizesUrl , {
            method: 'GET',
          })
          const res = await response.json();
        if (
          res &&
          res.sizes &&
          res.sizes.size &&
          res.sizes.size.length > 0
        ) {
          sizes = res.sizes.size;
        }
  }

</script>

<div class="slideshow-root">
  <div class="slideshow-background">
    <div class="slide" style="
    --fullecreenUrl: url({fullscreenUrl}); 
    --slideWidth: {slideWidth}px; 
    --slideHeight: {slideHeight}px; 
    --slideTop: {slideTop}px;
    --slideLeft: {slideLeft}px;
    ">
    </div>
  </div>
  <div class="icons-background">
    <div class="close-icon image-icon">
      <div on:click={() => {handleIconClick('close');}}> <!-- if you cant pass args to an on:x function by design, this is the way to do it. -->
        <Icon icon={faTimes} class="image-icon" title="Close" />
      </div>
    </div>
      {#if index > 0}
    <div class="left-icon image-icon">
      <div on:click={() => {handleIconClick(-1);}}>
        <Icon icon={faChevronLeft} class="image-icon" title={backArrowTitle} />
      </div>
    </div>
      {/if}
    <div class="right-icon image-icon">
      <div on:click={() => {handleIconClick(1);}}>
        <Icon icon={faChevronRight} class="image-icon" title={nextArrowTitle} />
      </div>
    </div>
  </div>
</div>


<style type="text/scss">
  .slideshow-root{
    top: 0;
    left: 0;
    z-index: 100;
    width: 100%;
    height: 100%;
    position: fixed;
    display: inline-block;
    box-sizing: border-box;

    .icons-background {
      top: 0;
      left: 0;
      z-index: 101;
      width: 100%;
      height: 100%;
      position: fixed;
      display: inline-block;
      box-sizing: border-box;

      .image-icon {
          position: relative;
          bottom: 0;
          font-size: 200%;
          text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;
          color: #ccc;
          border-radius: 5px;
          cursor: pointer;
          padding: 12px;
          margin: 3px;

          &:hover {
            color: white;
            border-color: white;
          }
        }
      .close-icon{
        visibility: visible;
        cursor: pointer;
        position: absolute;
        width: 50px;
        height: 50px;
        top: 0px;
        right: 0px;
        z-index: 200;
        color: white;
        padding: 15px;
        text-align: center;
        box-sizing: border-box;
        white-space: pre;
        display: flex;
        align-items: center;
        justify-content: center;
      }
      .left-icon{
        visibility: visible;
        cursor: pointer;
        position: absolute;
        width: 50px;
        height: 100%;
        left: 0%;
        color: white;
        padding: 15px;
        text-align: center;
        box-sizing: border-box;
        white-space: pre;
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 103;
      }
      .right-icon{
        visibility: visible;
        cursor: pointer;
        position: absolute;
        width: 50px;
        height: 100%;
        right: 0%;
        color: white;
        padding: 15px;
        text-align: center;
        
        box-sizing: border-box;
        white-space: pre;
        display: flex;
        align-items: center;
        justify-content: center;
        position: fixed;
        z-index: 102;
      }
    }

    .slideshow-background {
      background: rgba(0, 0, 0, 0.9);
      top: 0;
      left: 0;
      z-index: 100;
      width: 100%;
      height: 100%;
      display: inline-block;
      box-sizing: border-box;
      .slide{
        background-size: cover;
        background-position: center center;
        display: inline-block;
        vertical-align: center;
        box-sizing: border-box;
        position: absolute;
        background-image: var(--fullecreenUrl); 
                width: var(--slideWidth); 
                height: var(--slideHeight); 
                top: var(--slideTop);
                left: var(--slideLeft);
      }
    }
  }

  div :global(.image-icon) {
    font-size: 30px;
    vertical-align: middle;
    color: #ccc;
  }
  div :global(.image-icon):hover {
      color: white;
      border-color: white;
    }

</style>


