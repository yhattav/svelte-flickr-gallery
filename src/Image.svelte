<script>
	//name = 'arshole';
import Icon from 'fa-svelte'
import { faSync } from '@fortawesome/free-solid-svg-icons/faSync'
import { faTrash } from '@fortawesome/free-solid-svg-icons/faTrash'
import { faExpand } from '@fortawesome/free-solid-svg-icons/faExpand'
let icon = faSync;
export let index, id, large, large_Click, key, dto, imageSize, galleryWidth, delete_Click;
let rotate = 0, opacity = 1, backgroundImage = '', width = '100%', height = '100%', transform = '';

function urlFromDto(dto) {
    return `https://farm${dto.farm}.staticflickr.com/${dto.server}/${dto.id}_${dto.secret}.jpg`;
  }

function rotate_Click ()  {
    var rotateEdit = this.state.rotate;
    rotateEdit = rotateEdit + 90;
        this.setState({
      rotate: rotateEdit
    });
  }
</script>

<style>
.image-root {
  background-size: cover;
  background-position: center center;
  display: inline-block;
  vertical-align: top;
  box-sizing: border-box;
  position: relative;
  border: 1px solid white;
  background-color: red;
  width: var(--galleryImageSize);
  height: var(--galleryImageSize);
}
   .image-root div {
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

  .image-root div:hover div {
    visibility: visible;
  }

  .image-icon {
    font-size: 20px;
    vertical-align: middle;
    border: 1px solid #ccc;
    color: #ccc;
    border-radius: 5px;
    cursor: pointer;
    padding: 12px;
    margin: 3px;
  }
    .image-icon:hover {
      color: white;
      border-color: white;
    }
  

</style>

<div class="image-root" style="--galleryImageSize: {imageSize}px">
	<div
	style={{
		transform: 'rotate(-'+ rotate + 'deg)'
	}}>
	<div>
</div>
<!-- <Icon icon={icon}>
</Icon> -->
		<!-- <Icon icon=faSync /> -->
		<!-- class="image-icon rotateButton" id="rotateButton" name="sync-alt" title="rotate" onClick={() => rotate_Click()}<Icon icon=faTrash className="image-icon deleteButton" id="deleteButton" name="trash-alt" title="delete" onClick={() => this.props.deleteClick(this.props.id)}/> -->
		<!-- <Icon icon=faExpand className="image-icon expandButton" id="expandButton" name="expand" title="expand" onClick={() => this.props.largeClick(this.props.id)}/> -->
	</div>
</div>


