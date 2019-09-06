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
                                <input type="text" class="input" v-model="rules[c-1]" maxlength="1" v-if="c < 6">
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
            <div class="column is-4 is-offset-2">
                <div class="field is-horizontal">
                    <div class="field-label is-normal">
                        <label class="label">Test</label>
                    </div>
                    <div class="field-body">
                        <div class="field">
                            <div class="control">
                                <input :class="['input',{'is-danger': !match && test.length > 0,'is-success': match && test.length > 0}]" v-model="test" type="text" placeholder="Texto a verificar">
                            </div>
                            <p class="help is-danger">
                                {{result_message}}
                            </p>
                        </div>
                    </div>
                </div>
            </div>
            <div class="column is-2">
                <button class="button is-info" @click="testText">Validar texto</button>
            </div>
            <div class="column is-2">
                <button class="button is-primary" @click="generate">Generar Grafica</button>
            </div>
        </div>
        <div id="graph" class="container is-fluid" />
    </div>
</template>

<script>
import * as d3 from 'd3'
import 'd3-scale-chromatic'


const colorArray = ["#f44336", "#e91e63", "#9c27b0", "#673ab7", "#3f51b5", "#2196f3", "#03a9f4", "#00bcd4", "#009688", "#4caf50", "#8bc34a", "#cddc39", "#ffeb3b", "#ffc107", "#ff9800", "#ff5722", "#795548", "#9e9e9e", "#607d8b", "#e91e63", "#9c27b0", "#673ab7", "#3f51b5", "#2196f3", "#03a9f4", "#00bcd4", "#009688", "#4caf50", "#8bc34a", "#cddc39", "#ffeb3b", "#ffc107", "#ff9800"];

//TODO: en el mouse hover solo mostrar los links que tiene como source el node seleccionado
//TODO: subir más la posicion de los node, arcos grandes de abajo se cortan

export default {
    name: "Diagrama",
    data: () => ({
        title: "Diagrama de transicion de estados",
        columns: [1, 2, 3, 4, 5, 6],
        rows: [1, 2, 3, 4, 5, 6, 7],
        rules: ["a","b","","",""], 
        values: [
            [3, 4, 1, 1, 1, 1],
            [3, 4, 1, 1, 1, 0],
            [1, 0, 1, 1, 1, 0],
            [1, 1, 1, 1, 1, 1],
            [1, 1, 1, 1, 1, 1],
            [1, 1, 1, 1, 1, 1],
            [1, 1, 1, 1, 1, 1],
        ],
        nodes: [],
        links: [],
        match: false,
        result_message: '',
        test: ''
    }),
    methods: {
        generate() {
            d3.select("#graph").select("svg").remove();

            this.generateData();

            const data = {
                nodes: this.nodes,
                links: this.links
            }


            // set the dimensions and margins of the graph
            var
                width = document.getElementById("graph").clientWidth ,
                height = 750, 
                margin = {
                    top: height/8,
                    right: width/6,
                    bottom: 0,
                    left: width/6
                }

            // append the svg object to the body of the page
            var svg = d3.select("#graph")
                .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
                .append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ") scale(0.7 0.7)");

            var allNodes = data.nodes.map(function (d) {
                return d.name
            })

            //defines arrow
            svg.append("svg:defs")
                .append("svg:marker")
                .attr("id", "arrow")
                .attr("viewBox", "0 0 12 12")
                .attr("refX", 3)
                .attr("refY", 3)
                .attr("markerWidth", 12)
                .attr("markerHeight", 12)
                .attr("orient", "auto")
                .append("path")
                .attr("d", "M1,1 L5,3 L1,5 L3,3 L1,1")
                .style("fill", "#000")


            // A linear scale to position the nodes on the X axis
            var x = d3.scalePoint()
                .range([0, width])
                .domain(allNodes)

            // In my input data, links are provided between nodes -id-, NOT between node names.
            // So I have to do a link between this id and the name
            var idToNode = {};
            data.nodes.forEach(function (n) {
                idToNode[n.id] = n;
            });

            // Add the links
            var links = svg
                .selectAll('mylinks')
                .data(data.links)
                .enter()
                .append('path')
                .attr('d', function (d) {
                    let start = x(idToNode[d.source].name) // X position of start node on the X axis
                    let end = x(idToNode[d.target].name) // X position of end node

                    if (d.source === d.target) {

                        start -= 15
                        end += 15

                        return ['M', start, height - 210, // the arc starts at the coordinate x=start, y=height-30 (where the starting node is)
                                'C', // This means we're gonna build an elliptical arc
                                start, height - 410, ',',
                                end, height - 410, ',',
                                end, height - 210,
                            ] // We always want the arc on top. So if end is before start, putting 0 here turn the arc upside down.
                            .join(' ');
                    }

                    return ['M', start, height - 210, // the arc starts at the coordinate x=start, y=height-30 (where the starting node is)
                            'A', // This means we're gonna build an elliptical arc
                            ((start - end) / 4), ',', // Next 2 lines are the coordinates of the inflexion point. Height of this point is proportional with start - end distance
                            (start - end) / 4, 0, 0, ',',
                            data.links.find(item => item.source == d.target && item.target == d.source) ? 1 : 0, end, ',', height - 210
                        ] // We always want the arc on top. So if end is before start, putting 0 here turn the arc upside down.
                        .join(' ');
                })
                .style("fill", "none")
                .attr("stroke", function (d, i) {
                    return colorArray[i]
                })
                .style("stroke-width", 4)
                .attr("marker-start", "url(#arrow)")
                .attr("marker-mid", "url(#arrow)")
                .attr("marker-end", "url(#arrow)");

            // Add the circle for the nodes
            var nodes = svg
                .selectAll("mynodes")
                .data(data.nodes.sort(function (a, b) {
                    return +b.n - +a.n
                }))
                .enter()
                .append("g")
                .attr("transform", function () { return "translate(0,-180)" })

            nodes
                .append("circle")
                .attr("cx", function (d) {
                    return (x(d.name))
                })
                .attr("cy", height - 30)
                .attr("r", 20)
                .style("fill", "#005cb2")
                .style("stroke", function (d) {
                    return d.accept ? "6ab7ff" : null
                })
                .style("stroke-width", function (d) {
                    return d.accept ? 5 : 0
                })
            nodes
                .append("text")
                .text(function (d) {
                    return (d.name)
                })
                .style("text-anchor", "end")
                .attr("transform", function (d) {
                    return ("translate(" + (x(d.name) + 8) + "," + (height - 30) + ")rotate(-45)")
                })
                .style("fill", "#fff")
                .style("font-size", 12)


            // Add the highlighting functionality
            nodes
                .on('mouseover', function (d) {
                    // Highlight the nodes: every node is green except of him
                    nodes
                        .style('opacity', .2)
                    d3.select(this)
                        .style('opacity', 1)
                    // Highlight the connections
                    links
                        .style('stroke-opacity', function (link_d) {
                            return link_d.source === d.id ? 1 : .2;
                        })
                        .style('stroke-width', function (link_d) {
                            return link_d.source === d.id ? 4 : 1;
                        })


                })
                .on('mouseout', function () {
                    nodes.style('opacity', 1)
                    links
                        .style("stroke-width", 4)
                        .style("stroke-opacity", 1)

                })


        },
        generateLinks() {
            let links = [];
            let row;

            for (let i = 0; i < this.values.length; i++) {

                row = this.values[i]

                for (let q = 0; q < row.length; q++) {

                    if (row[q] > 1) {

                        links.push({
                            source: i + 1,
                            target: row[q] - 1,
                        })

                    }

                }
            }

            this.links = links
            
            /*.reduce((acc, current) => {
                const x = acc.find(item => item.target === current.target && item.source === current.source);
                if (!x) {
                    return acc.concat([current]);
                } else {
                    return acc;
                }
            }, [])*/
        },
        generateNodes() {
            let nodes = [];
            let value;

            for (let r = 0; r < this.rows.length; r++) {

                value = this.rows[r];
                if (this.liniksHasNode(value))
                    nodes.push({
                        id: value,
                        name: `Q${value}`,
                        accept: this.values[r][5] == 0
                    });

            }

            this.nodes = nodes;
        },
        liniksHasNode(node) {

            for (let i in this.links) {

                if (this.links[i].target == node || this.links[i].source == node) return true

            }

            return false
        },
        generateData() {
            this.generateLinks()
            this.generateNodes()
        },
        testText(){
            
            let correct_info = this.values.length > 0


            if(correct_info){

                this.result_message = ""
                let charts = this.test.split("")
                let row,rule_index,row_index=0

                
                for(let i in charts){
                    row = this.values[row_index]

                    rule_index = this.rules.indexOf(charts[i])

                    if( rule_index != -1){
                        
                        this.match = row[rule_index] == 0 || (charts.length -1 == i && row[5] == 0)

                        if( row[rule_index] - 2 > 0 ) row_index = row[rule_index] - 2
                        
                    }else{
                        
                        this.match=false
                    }
                    
                }

                return;
            }
            
            this.match = false

            this.result_message = "Parametros incompletos"
        }
    },
}
</script>