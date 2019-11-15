<script>
  import Icon from 'fa-svelte'
  import { faSync } from '@fortawesome/free-solid-svg-icons/faSync'
  import { faTrash } from '@fortawesome/free-solid-svg-icons/faTrash'
  import { faExpand } from '@fortawesome/free-solid-svg-icons/faExpand'
  import { onMount, beforeUpdate, afterUpdate} from 'svelte';

  export let index, id, large, handleFullscreen, key, dto, imageSize, galleryWidth, deleteClick, rotate = 0, handleRotate;

  let icon = faSync;
  let imageRotate, opacity = 1, backgroundImage = '', width = '100%', height = '100%', transform = '', imageUrl;

  $: imageUrl = `https://farm${dto.farm}.staticflickr.com/${dto.server}/${dto.id}_${dto.secret}.jpg`;
  $: imageRotate = rotate ? rotate : 0;

  function handleRotateClick ()  {
    handleRotate(index, imageRotate+90);
  }

  function deleteImage()  {
    deleteClick(id);
  }
  function expandImage()  {
    handleFullscreen(id);
  }

</script>

<div class="image-root" style="--imageUrl: url({imageUrl}); --galleryImageSize: {imageSize}px; --imageTransform: rotate({imageRotate}deg)">
	<div class="image-icon-box" style="transform: rotate(-{imageRotate}deg);">
    <div class="image-icon-wrapper rotateButton" title="rotate" on:click={handleRotateClick}>
      <Icon class="image-icon" icon={faSync}/>
    </div>
    <div class="image-icon-wrapper deleteButton" title="delete" on:click={deleteImage}>
      <Icon icon={faTrash} class="image-icon" />
    </div>
    <div class="image-icon-wrapper expandButton" title="expand" on:click={expandImage}>
      <Icon icon={faExpand} class="image-icon"/>
    </div>
	</div>
</div>


<style>
.image-root {
  background-size: cover;
  background-position: center center;
  display: inline-block;
  vertical-align: top;
  box-sizing: border-box;
  position: relative;
  border: 1px solid white;
  width: var(--galleryImageSize);
  height: var(--galleryImageSize);
  background-image: var(--imageUrl);
  transform: var(--imageTransform);
  }

  .image-root .image-icon-box {
  visibility: hidden;
  background: rgba(0, 0, 0, 0.7);
  cursor: pointer;
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  color: white;
  padding: 15px;
  text-align: center;
  text-align: center;
  box-sizing: border-box;
  white-space: pre;
  display: flex;
  align-items: center;
  justify-content: center;
}

.image-root:hover div {
  visibility: visible;
}
.image-icon-wrapper{
  vertical-align: middle;
  border: 1px solid #ccc;
  border-radius: 5px;
  cursor: pointer;
  padding: 12px;
  margin: 3px;
}
div :global(.image-icon) {
  font-size: 20px;
  vertical-align: middle;
  color: #ccc;
}
div :global(.image-icon):hover {
    color: white;
    border-color: white;
  }

</style>


