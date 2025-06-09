<script>
    import {  } from 'svelte/reactivity/window';
    import { T, useTask, extend, useThrelte } from "@threlte/core";
    import {
        useGltf,
        useDraco,
        OrbitControls,
        TransformControls,
        useGltfAnimations,
        Gizmo
    } from "@threlte/extras";
    import * as THREE from 'three';
    import { interactivity } from '@threlte/extras';
    interactivity()

    const dracoLoader = useDraco();
    const gltf = useGltf("/models/LittlestTokyo.glb", { dracoLoader });
    const { actions, mixer } = useGltfAnimations(gltf);

    $effect(() => {
        $actions["Take 001"]?.play();
        mixer.timeScale = 0.5;
    });

    // gltf.then((gltf)=> console.log(gltf.scene))
    
    let camera = $state(null);

    const position = $state({x:-0.29, y:-1.04,z:2.10})

    // Camera path points
    const controlPoints = $state([
        new THREE.Vector3(-0.28, -0.53, 3.30), 
        new THREE.Vector3(-0.30, -0.88, 2.68), 
        new THREE.Vector3(-0.31, -1.05, 1.59), 
        new THREE.Vector3(-0.36, -1.05, 0.32), 
        new THREE.Vector3(0.02, -1.05, 0.03), 
        new THREE.Vector3(1.22, -1.05, -0.85), 
        new THREE.Vector3(1.25, -1.05, 0.33), 
        new THREE.Vector3(1.15, -1.07, 1.66)
    ]);

    // Add lookAt targets for each control point
    const lookAtTargets = $state([
        new THREE.Vector3(-0.29, -0.20, 2.65), 
        new THREE.Vector3(-0.63, -0.88, 2.09), 
        new THREE.Vector3(-0.06, -1.05, 0.93), 
        new THREE.Vector3(-0.06, -1.08, -0.23), 
        new THREE.Vector3(0.57, -1.16, -0.09), 
        new THREE.Vector3(1.00, -1.03, -0.30), 
        new THREE.Vector3(0.93, -1.08, 0.68), 
        new THREE.Vector3(0.61, -1.05, 1.33)
    ]);

    // Create curve for lookAt targets too
    let lookAtCurve = $derived(new THREE.CatmullRomCurve3(lookAtTargets));

    // Create curve
    let curve = $derived(new THREE.CatmullRomCurve3(controlPoints));

    let scrollProgress = $state(0);


    // Update scroll on wheel
    function handleWheel(event) {
        scrollProgress = Math.max(0, Math.min(1, scrollProgress + event.deltaY * 0.00009));
    }

    let targetPosition = $derived(curve.getPoint(scrollProgress));
    let smoothCameraPosition = $state(controlPoints[0].clone());

    let smoothLookAt = $state(lookAtTargets[0].clone());
    let finalLookAt = $state(lookAtTargets[0].clone());

    useTask(() => {
        const lerpFactor = 0.05; // Adjust for smoothness
        
        smoothCameraPosition = new THREE.Vector3(
            smoothCameraPosition.x + (targetPosition.x - smoothCameraPosition.x) * lerpFactor,
            smoothCameraPosition.y + (targetPosition.y - smoothCameraPosition.y) * lerpFactor,
            smoothCameraPosition.z + (targetPosition.z - smoothCameraPosition.z) * lerpFactor
        );

        // Get interpolated lookAt target from the curve
        const targetLookAtPoint = lookAtCurve.getPoint(scrollProgress);
        
        // // Smooth the lookAt
        smoothLookAt = new THREE.Vector3(
            smoothLookAt.x + (targetLookAtPoint.x - smoothLookAt.x) * lerpFactor,
            smoothLookAt.y + (targetLookAtPoint.y - smoothLookAt.y) * lerpFactor,
            smoothLookAt.z + (targetLookAtPoint.z - smoothLookAt.z) * lerpFactor
        );

        // Apply mouse delta to the lookAt target
        finalLookAt = new THREE.Vector3(
            smoothLookAt.x + mouseDelta.x * mouseInfluence,
            smoothLookAt.y + mouseDelta.y * mouseInfluence,
            smoothLookAt.z
        );
        
        if (camera) {
            camera.lookAt(finalLookAt);
        }
    
    });


    // Mouse movement
    let mouseX = $state(0);
    let mouseY = $state(0);
    let mouseDelta = $state({ x: 0, y: 0 });

    // Mouse sensitivity settings
    const mouseSensitivity = $state(0.001); // Adjust this value to control sensitivity
    const mouseInfluence = $state(0.9); // How much mouse movement affects lookAt (0-1)

    function handleMouseMove(event) {
        // Convert mouse position to normalized coordinates (-1 to 1)
        const normalizedX = (event.clientX / window.innerWidth) * 2 - 1;
        const normalizedY = -(event.clientY / window.innerHeight) * 2 + 1;
        
        mouseX = normalizedX;
        mouseY = normalizedY;
        
        // Calculate delta for lookAt offset
        mouseDelta = {
            x: normalizedX * mouseSensitivity * 10,
            y: normalizedY * mouseSensitivity * 10
        };
    }
    
    // temp for dev---------------------

    // Visualize lookAt curve
    let lookAtCurvePoints = $derived(lookAtCurve.getPoints(180));
    let lookAtCurveGeometry = $derived(new THREE.BufferGeometry().setFromPoints(lookAtCurvePoints));

    // Update lookAt point
    function updateLookAtPoint(index, position) {
        lookAtTargets[index] = new THREE.Vector3(position.x, position.y, position.z);
    }

    // Visualize curve
    let curvePoints = $derived(curve.getPoints(180));
    let curveGeometry = $derived(new THREE.BufferGeometry().setFromPoints(curvePoints));
    
    // Update control point
    function updatePoint(index, position) {
        controlPoints[index] = new THREE.Vector3(position.x, position.y, position.z);
    }
    
    $effect(()=>{
        // $inspect(controlPoints)
        console.log("control points \n",
            controlPoints.map(point => 
            `new THREE.Vector3(${point.x.toFixed(2)}, ${point.y.toFixed(2)}, ${point.z.toFixed(2)})`
            ).join(", \n")
        )
        console.log("look at points \n",
            lookAtTargets.map(point => 
            `new THREE.Vector3(${point.x.toFixed(2)}, ${point.y.toFixed(2)}, ${point.z.toFixed(2)})`
            ).join(", \n")
        )
    })
    
    // gizmo
    let type = $state('sphere')
    let speed = $state(1)
    let placement= $state('bottom-left')
    let size = $state(86)
    let left = $state(10)
    let top = $state(10)
    let right = $state(10)
    let bottom = $state(10)
    let center = $state([0, 0, 0])

    let hoveredIndex = $state(null);
    let hoveredIndexCam = $state(null);
</script>

<svelte:window on:wheel={handleWheel} on:mousemove={handleMouseMove} />




<T.PerspectiveCamera 
    makeDefault
    fov={60} 
    bind:ref={camera}
    position={[smoothCameraPosition.x, smoothCameraPosition.y, smoothCameraPosition.z]}
    >     
</T.PerspectiveCamera>

{#await gltf then { scene }}
    <!-- {console.log('GLTF loaded:', {scene})} -->
    <T is={scene}  
    position={[0,0,0]}
    scale={0.007} />
{/await}


<!-- temp dev ------------------------------------------- -->

<!-- <T.PerspectiveCamera 
    makeDefault
    fov={60} 
    position={[0,0,3]}
    >     
    <OrbitControls>
         <Gizmo
          {type}
          {speed}
          {placement}
          {size}
          offset={{
            top,
            left,
            bottom,
            right
          }}
        />
    </OrbitControls>
</T.PerspectiveCamera> -->


<!-- Visualize lookAt curve -->
<!-- <T.Line geometry={lookAtCurveGeometry}>
    <T.LineBasicMaterial color="blue" />
</T.Line>

{#each lookAtTargets as target, i}
    <T.Mesh 
        position={[target.x, target.y, target.z]}
        onclick={() =>{ hoveredIndexCam = null, hoveredIndex == i ?  hoveredIndex = null:hoveredIndex = i} }
    >
        {#snippet children({ ref })}
            <T.SphereGeometry args={[0.08]} />
            <T.MeshBasicMaterial color="blue" />
            {#if hoveredIndex === i}
                <TransformControls 
                    object={ref}
                    
                    onobjectChange={(event) => updateLookAtPoint(i, ref.position)}
                />
            {/if}
        {/snippet}
    </T.Mesh>
{/each} -->

<!-- Connection lines from camera to lookAt points -->
<!-- {#each controlPoints as camPoint, i}
    <T.Line>
        <T.BufferGeometry>
            <T.BufferAttribute 
                attach="attributes.position"
                array={new Float32Array([
                    camPoint.x, camPoint.y, camPoint.z,
                    lookAtTargets[i].x, lookAtTargets[i].y, lookAtTargets[i].z
                ])}
                count={2}
                itemSize={1}
            />
        </T.BufferGeometry>
        <T.LineBasicMaterial color="yellow" />
    </T.Line>
{/each} -->


<!-- <T.Line geometry={curveGeometry}>
    <T.LineBasicMaterial color="red" />
</T.Line>

{#each controlPoints as point, i}
    <T.Mesh position={[point.x, point.y, point.z]}
    onclick={() =>{hoveredIndex = null , hoveredIndexCam == i ?  hoveredIndexCam = null:hoveredIndexCam = i }}    
    >
        {#snippet children({ ref })}
        
        <T.SphereGeometry args={[0.05]} />
        <T.MeshBasicMaterial color="red" />
            {#if hoveredIndexCam === i}
                <TransformControls 
                    object = {ref}          
                    onobjectChange ={(event) => updatePoint(i, ref.position)}
                />
            {/if}
        {/snippet}
    </T.Mesh>
{/each} -->



