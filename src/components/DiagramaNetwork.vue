
<template>
    <div class="container">
        <div class="columns">
            <div class="column is-2">
            </div>
            <div class="column is-half">
                <table class="table">
                    <thead>
                        <tr>
                            <th>-</th>
                            <th v-for="c in columns" :key="'key-'+c">
                                <input type="text" class="input" v-if="c < 6">
                            </th>
                            <th><abbr title="FDC">FDC</abbr></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(r,ri) in rows" :key="'r'+r">
                            <th>Q{{r}}</th>
                            <th v-for="(c,ci) in columns" :key="'i-'+r+c">
                                <div class="select" v-if="c < 6">
                                    <select v-model="values[ri][ci]">
                                        <option :value="0">OK</option>
                                        <option :value="1">ERROR</option>
                                        <option :value="2">Q1</option>
                                        <option :value="3">Q2</option>
                                        <option :value="4">Q3</option>
                                        <option :value="5">Q4</option>
                                        <option :value="6">Q5</option>
                                        <option :value="7">Q6</option>
                                        <option :value="8">Q7</option>
                                    </select>
                                </div>
                            </th>
                            <th>
                                <div class="select">
                                    <select v-model="values[ri][5]">
                                        <option :value="0">OK</option>
                                        <option :value="1">ERROR</option>
                                    </select>
                                </div>
                            </th>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="column is-2">
            </div>
        </div>
        <div class="columns">
            <div class="column is-2 is-offset-2">
                <button class="button is-primary" @click="generate">Generar</button>
            </div>
        </div>
        <div id="graph" class="container is-fluid" />
    </div>
</template>

<script>

import * as d3 from 'd3'

const colorArray = ["#f44336", "#e91e63", "#9c27b0", "#673ab7", "#3f51b5", "#2196f3", "#03a9f4", "#00bcd4", "#009688", "#4caf50", "#8bc34a", "#cddc39", "#ffeb3b", "#ffc107", "#ff9800", "#ff5722", "#795548", "#9e9e9e", "#607d8b", "#e91e63", "#9c27b0", "#673ab7", "#3f51b5", "#2196f3", "#03a9f4", "#00bcd4", "#009688", "#4caf50", "#8bc34a", "#cddc39", "#ffeb3b", "#ffc107", "#ff9800"];

export default {
    name: "Diagrama",
    data: ()=>({
        title: "Diagrama de transicion de estados",
        columns: [1,2,3,4,5,6],
        rows: [1,2,3,4,5,6,7],
        values: [
            [2,3,1,1,1,1],
            [4,4,1,1,1,0],
            [1,1,5,5,0,1],
            [1,1,1,6,1,0],
            [1,1,7,7,7,1],
            [1,1,1,8,8,0],
            [1,1,1,4,5,1],
        ]
    }),
    methods: {
        generate(){
            d3.select("#graph").select("svg").remove();

            const data = {
                nodes: this.generateNodes(),
                links: this.generateLinks()
            }


            // set the dimensions and margins of the graph
            var margin = {top: 10, right: 30, bottom: 30, left: 40},
            width = document.getElementById("graph").clientWidth - margin.left - margin.right,
            height = 600 - margin.top - margin.bottom;

            // append the svg object to the body of the page
            var svg = d3.select("#graph")
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform",
                    "translate(" + margin.left + "," + margin.top + ")");

            // Initialize the links
            var link = svg
                .selectAll("line")
                .data(data.links)
                .enter()
                .append("line")
                .style("stroke", function(d,i){return colorArray[i]})
                .style("stroke-width", "3")

            // Initialize the nodes
            var node = svg
                .selectAll("g")
                .data(data.nodes)
                .enter()
                .append("g")
                
            node 
                .append("circle")
                .attr("r", 20)
                .style("fill", "#005cb2")
                .style("stroke", function(d){ return d.accept ? "6ab7ff" : null })
                .style("stroke-width", function(d){ return d.accept ? 5 : 0 })
            node
                .append("text")
                .attr("dx", function(){return -10})
                .attr("dy", function(){return 5})
                .style('fill', '#fff')
                .text(function(d){return d.name})

            // Let's list the force we wanna apply on the network
            d3.forceSimulation(data.nodes)                 // Force algorithm is applied to data.nodes
                .force("link", d3.forceLink()                               // This force provides links between nodes
                        .id(function(d) { return d.id; })                     // This provide  the id of a node
                        .links(data.links)                                    // and this the list of links
                        .distance(100)
                        .strength(0.1)
                )
                .force("charge", d3.forceManyBody().strength(-800))         // This adds repulsion between nodes. Play with the -400 for the repulsion strength
                .force("center", d3.forceCenter(width / 2, height / 2))     // This force attracts nodes to the center of the svg area
                .force("yAxis",d3.forceY(height/2))
                .on("end", ticked);

            // This function is run at each iteration of the force algorithm, updating the nodes position.
            function ticked() {
                link
                    .attr("x1", function(d) { return d.source.x; })
                    .attr("y1", function(d) { return d.source.y; })
                    .attr("x2", function(d) { return d.target.x; })
                    .attr("y2", function(d) { return d.target.y; });

                node
                    .attr("transform", function (d) { return `translate(${d.x+6},${d.y-6})` })
            }


        },
        generateNodes(){
            let nodes = [];
            let value;

            for(let r = 0; r < this.rows.length; r++){
                
                value = this.rows[r];
                nodes.push( { 
                    id: value, 
                    name: `Q${value}`,
                    accept: this.values[r][5] == 0
                } );

            }

            return nodes;
        },
        generateLinks(){
            let links = [];
            let row;

            for(let i = 0; i < this.values.length; i++){

                row = this.values[i]

                for(let q = 0; q < row.length; q++){

                    if(row[q] > 1){
                        
                        links.push({
                            source: i+1,
                            target: row[q]-1
                        })

                    }

                }
            }


            return links;
        }
    },
}
</script>