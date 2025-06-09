<script>
    import { Canvas } from '@threlte/core'
    // import Scene from '../lib/scene.svelte'
    import Scene from "../threeComponents/scenes/scene.svelte"
    import { NoToneMapping } from 'three';
    import { Sky } from '@threlte/extras';

</script>

<div class="h-screen fixed left-0 top-0 w-full bg-[#202020]">
    <Canvas toneMapping ={NoToneMapping}>
        <Sky elevation={8}/>
        <Scene/>
    </Canvas>
</div>