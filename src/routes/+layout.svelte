<script>
    import ThreeDSpace from "../lib/threeComponents/ThreeDSpace.svelte"

    let { children } = $props();
    import "../app.css";
</script>


<ThreeDSpace/>

{@render children()}
