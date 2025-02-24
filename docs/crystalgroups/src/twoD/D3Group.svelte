<script>
	import { onMount } from 'svelte';
    import {GroupElement, FiniteGroup} from "./groupmath.js";
    import GroupElementDisplay from "./GroupElementDisplay.svelte";
    import SVGCayleyArrows from "./SVGCayleyArrows.svelte";

    import D3ElementCanvas from "./D3ElementCanvas.svelte";

    import { chooseElementBorderColor} from "../colors.js";

    let r = new GroupElement("r", "(123)");
    let f = new GroupElement("f", "(23)");
    export let D3group = new FiniteGroup([r,f], {"rfr":"f", "rrr":"", "ff":""});

    export let isElementVisible = D3group.elements.map(element => true);

    let generators = ["r", "f"];

    //placing them on a 2D grid

    export let positions = new Map();
    D3group.elements.forEach(element => {positions.set(element, [0,0])}) //fill this dict with one position per element

    let startPos = [12,12];
    let outerRadius = 10;
    let innerRadius = 4;
    let rotationRadians = 120 * Math.PI / 180;
    let startRadians = -Math.PI/2; //start upwards

    let currentElem = D3group.getElemByName("e");
    for(let i=0;i<3;i++){
        //place rotation elements on the outside of the circle.
        let position = [startPos[0] + outerRadius * Math.cos(rotationRadians * i + startRadians), 
                        startPos[1] + outerRadius * Math.sin(rotationRadians * i + startRadians)];
        positions.set(currentElem, position)

        //place flip elements on the inside of the circle
        let flipElem = D3group.multiply(D3group.getElemByName("f"), currentElem);
        let flipPosition = [startPos[0] + innerRadius * Math.cos(-rotationRadians * i + startRadians), //rotation reversed
                           startPos[1] + innerRadius * Math.sin(-rotationRadians * i + startRadians)];
        positions.set(flipElem, flipPosition)

        //move to next element
        currentElem = D3group.multiply(D3group.getElemByName("r"), currentElem);
    }


    function chooseBorderColor(element){
        if(element.name == "e"){
            return identityColor;
        }
        if(D3group.generators.indexOf(element) !== -1){ //if the element is a generator, color it appropriately
            return generatorColors[D3group.generators.indexOf(element)] ;
        }
        return defaultGroupElementBorderColor;
    }

    let defaultArrowVisibility = {};
    D3group.elements.forEach(startElement => {
                defaultArrowVisibility[startElement.name] = D3group.generators.map(generator => true) //every generator starts false
            }
    )
    defaultArrowVisibility["e"] = [true, true];
    export let isArrowVisibleMap = defaultArrowVisibility; //elementTimesGenerators[startElem.name][generator_index] is true if the arrow starting from startElemn and corresponding to the ith generator should show an arrow
    

    export let elementsWhoseNamesNotToShow = [];
</script>

<style>
/*
.groupdisplay{
    display: grid;
    grid: 1fr / 1fr;
}
.ontop{
    grid-column-start: 1;
  grid-row-start: 1;
}*/
</style>

<div class="groupdisplay">
    {#each D3group.elements as element, i}
        {#if isElementVisible[i]}
            <GroupElementDisplay element={element} showElementName={elementsWhoseNamesNotToShow.indexOf(element.name) == -1}
            borderColor={chooseElementBorderColor(D3group, element)}
            top={positions.get(element)[1]} left={positions.get(element)[0]}
            >
                <D3ElementCanvas element={element}/>
            </GroupElementDisplay>
        {/if}
    {/each}

    <SVGCayleyArrows group={D3group} positionsPerElementMap={positions} isArrowVisibleMap={isArrowVisibleMap}/>
</div>
