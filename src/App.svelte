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

    function setPos(event) {
        const { top, left } = refs.container.getBoundingClientRect();
        let x = event.clientX - left, y = event.clientY - top
        console.log('setPos');
        let elt = coordsElt()
        console.log(elt)
        console.log(elt.parentElement)
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
        if (typeof evtNm == 'string' ) {
            evtAry = [evtNm]
        }
        for (const evtNm of evtAry ) {
            action(evtNm, handler, false)
        }

    }

    function drag(node, callback) {
        console.log('drag');
        
        const onDown = event => {
            if (event.type == 'mousedown' && event.which !== 1) return;

            event.preventDefault();

            dragging = true;

            const onUp = () => {
                let elt = coordsElt()
                elt.style.display = 'none'

               dragging = false;

                listener(window, 'rm', [ 'touchmove', 'mousemove' ], callback)
                listener(window, 'rm', [ 'touchend',  'mouseup'   ], callback)
                // window.removeEventListener('mousemove', callback, false);
                // window.removeEventListener('mouseup', onmouseup, false);
            };
            listener(window, 'add',['touchmove', 'mousemove'], callback )
            listener(window, 'add',['touchmove', 'mousemove'], onUp )
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




<h3>hi</h3>
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
