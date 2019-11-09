<div>
  <Icon class="myClass1 myClass2" icon={faCircle}>
  </Icon>
</div>

<script>
import Icon from 'fa-svelte'
import { faCircle } from '@fortawesome/free-solid-svg-icons/faCircle'

let icon = faCircle;
</script>
<style>

div :global(.myClass1) {
    color: red;
}

div :global(.myClass2) {
    font-size: 14px;
}
</style>