<script lang="ts">
	import { dndzone} from 'svelte-dnd-action';
    import GridItem from 'svelte-web-components/layout/GridItem.svelte';
    import layout, { Tab } from '../../generated/layout';
    import { Writable } from "svelte/store";
    import {onMount} from "svelte"
    import "gridstack/dist/gridstack.min.css";
    import "./gs.css"

// Import GridStack 3.2.0
import GridStack from "gridstack/dist/gridstack-all";
import "gridstack/dist/h5/gridstack-dd-native"; //  HTML5 drag&drop
    import { subStore } from 'immer-loves-svelte';
    import { widgetDefinitions } from '../../generated/elements';
    export let tab : Writable<Tab>;
    export let selectedIndex = 0;
	
	let idx = 0;
    let elements = subStore(tab, t=>t.elements)
    let items = [];
    Object.assign(window, {select: (newIndex) => { console.log(newIndex); selectedIndex = newIndex}});
    $: {
        // On change to the elements array, recreate the itemlist for the selectionlayer grid.
        items = $elements.map((element, i) => (
        {
            x: element.layout.x - 1,
            y: element.layout.y - 1,
            h: element.layout.height,
            w: element.layout.width,
            id:element.id, locked:true,
            minW:widgetDefinitions[element.type]?.config.layout?.minWidth,
            minH:widgetDefinitions[element.type]?.config.layout?.minHeight,
            content:`<div onclick=\"select(${i})\" style=\"background:rgba(77, 77, 110, 0.4); height:100%; width:100%\"></div>`}
        ));
        if (grid !== null) {
            console.log(items)
            grid.load(items, true)
        }
    }

    let layoutStore = {}
    $: $elements.forEach((element, i) => (
        layoutStore[element.id] = 
        {x: subStore(tab, t=>t.elements[i].layout.x),
        y: subStore(tab, t=>t.elements[i].layout.y),
        h: subStore(tab, t=>t.elements[i].layout.height),
        w: subStore(tab, t=>t.elements[i].layout.width)}
    ));
	
let rows = 9;
let cols = 12;

let grid = null;

let widgets = []



onMount(()=>{
        grid = GridStack.init({
        cellHeight: "50px",
        cellWidth: "50px",
        column:50,
        float: true,
        disableOneColumnMode: true,
        margin:0,
      })
        grid.on("dragstop", function (event, element) {
                const node = element.gridstackNode;
                console.log(`you just dragged node #${node.id} to ${node.x},${node.y} – good job!`);

                let {x, y, w, h} = layoutStore[node.id]
                x.set(node.x + 1);
                y.set(node.y + 1);
                });
        grid.on("resizestop", function (event, element) {
                const node = element.gridstackNode;
                console.log(`you just resized node #${node.id} to ${node.w},${node.h} – good job!`);

                let {x, y, w, h} = layoutStore[node.id]
                w.set(node.w);
                h.set(node.h);
                });
        console.log(items)
        grid.load(items, true)

        // In GridStack 3 we use "w" instead of "width"
        widgets.forEach((pair) => {
            console.log(pair)
            //const widget = grid.addWidget(pair[0], pair[1]);
            // // We set an id for this widget
            // widget.id = pair[1].id;
            // // Then we append it manually to GridStack html element
            // grid.el.appendChild(widget);
            // // And then - we inform GridStack, that a new element is added.
            // grid.makeWidget(`#${pair[1].id}`);
        });

        });
        
</script>
<div class="grid-stack" style="min-height:100%"></div>

