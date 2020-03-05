<script>
    import * as yootils from 'yootils';
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

//	export let name;
    let type     = 'vertical', h, w, size, pos;
    let dragging = false
    let refs = {}

    let table = [ 1, 2, 3, 4]

    // Code adapted from SplitPane in the REPL
    function coordsElt() { return document.getElementById('coords') 	}

    function setPos(e) {
        function clientX() { return e.ClientX ? e.ClientX : e.touches[0].clientX }
        function clientY() { return e.ClientY ? e.ClientY : e.touches[0].clientY }
        
        const { top, left } = refs.container.getBoundingClientRect();
        let x = clientX() - left, y = clientY() - top
        let elt = coordsElt()
        console.log('stopPropagation');
        
        e.stopPropagation()
        elt.style.display = 'block'
        elt.style.left = x + 'px'
        elt.style.top = y  + 'px'
        elt.innerHTML = x + ' ' + y

        const px = type === 'vertical' ? y : x
        pos = 100 * px / size;
//		dispatch('change');
    }

    function listener(node, act, evtNm, handler) {
        let evtAry
        let action = act == 'rm' ? node.removeEventListener : node.addEventListener
        evtAry = typeof evtNm == 'string' ? [evtNm] : evtNm
        for (const evtNm of evtAry ) {
            action(evtNm, handler, false)
        }
    }

    function drag(node, callback) {
        const onDown = event => {
            if (event.type == 'mousedown' && event.which !== 1) return;

            event.preventDefault();

            dragging = true;

            const onUp = () => {
                let elt = coordsElt()
                elt.style.display = 'none'

               dragging = false;

                listener(node, 'rm', [ 'touchmove', 'mousemove' ], callback)
                listener(node, 'rm', [ 'touchend',  'mouseup'   ], onUp)
                // window.removeEventListener('mousemove', callback, false);
                // window.removeEventListener('mouseup', onmouseup, false);
            };
            listener(document.body, 'add', 'touchmove', 
                (e) => { console.log('body'); e.stopPropagation()} )
            listener(node, 'add',['touchmove', 'mousemove'], callback )
            listener(node, 'add',['touchend', 'mouseup'],    onUp )
            // window.addEventListener('mousemove', callback, false);
            // window.addEventListener('mouseup', onmouseup, false);
        }

        // node.addEventListener('mousedown', mousedown, false);
        listener(node, 'add', ['touchstart', 'mousedown'], onDown)

        return {
            destroy() {
                // node.removeEventListener('mousedown', onmousedown, false);
                listener(node, 'rm', 'mousedown', onDown)
            }
        };
    }

    $: side = type === 'horizontal' ? 'left' : 'top';
    $: dimension = type === 'horizontal' ? 'width' : 'height';

</script>



<div class="container">
    <table  bind:this={refs.container} bind:clientWidth={w} bind:clientHeight={h}>
        {#each table as item, i}
            <tr>
                <td>
                {item} {i}
                {#if i != table.length -1 }
                   <hr use:drag={setPos}/>
                {/if}	
                </td>
            </tr>
        {/each}
    </table>
    <div id="coords"></div>
</div>

<style>

    .container {
        position: relative;
        display: block;
        border: 1px solid black;
    }
    hr { 
        border-color:red;
        cursor: crosshair
    }
    /* main {
        text-align: center;
        padding: 1em;
        max-width: 240px;
        margin: 0 auto;
    } */

    /* h1 {
        color: #ff3e00;
        text-transform: uppercase;
        font-size: 4em;
        font-weight: 100;
    } */

    /* @media (min-width: 640px) {
        main {
            max-width: none;
        }
    } */
    #coords { 
        display: none;
        position: absolute;
        min-height: 1em;
        min-width: 100em;
        overflow: visible;
        color: blue;
        background-color: yellow;
    }
</style>
