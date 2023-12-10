# petitpapillon
get the chance to win
var padding = { top: 20, right: 40, bottom: 0, left: 0 },
  w = 500 - padding.left - padding.right,
  h = 500 - padding.top - padding.bottom,
  r = Math.min(w, h) / 2,
  rotation = 0,
  oldrotation = 0,
  picked = 100000,
  oldpick = [],
  color = d3.scale.ordinal().range(["pink", "blue", "purple"]),
  data = [
    { label: "nothing", value: 1, question: "What CSS property is used for specifying the area between the content and its border?" }, // padding
    { label: "IMAC PRO", value: 2, question: "What CSS property is used for changing the font?" }, // font-family
    { label: "SUZUKI", value: 3, question: "What CSS property is used for changing the color of text?" }, // color
  ],
  svg = d3
    .select("#chart")
    .append("svg")
    .data([data])
    .attr("width", w + padding.left + padding.right)
    .attr("height", h + padding.top + padding.bottom),
  container = svg
    .append("g")
    .attr("class", "chartholder")
    .attr("transform", "translate(" + (w / 2 + padding.left) + "," + (h / 2 + padding.top) + ")"),
  vis = container.append("g");

var pie = d3.layout.pie().sort(null).value(function (d) {
  return 1;
});
var arc = d3.svg.arc().outerRadius(r);

var arcs = vis.selectAll("g.slice").data(pie).enter().append("g").attr("class", "slice");

arcs
  .append("path")
  .attr("fill", function (d, i) {
    return color(i);
  })
  .attr("d", function (d) {
    return arc(d);
  });

arcs
  .append("text")
  .attr("transform", function (d) {
    d.innerRadius = 0;
    d.outerRadius = r;
    d.angle = (d.startAngle + d.endAngle) / 2;
    return "rotate(" + (d.angle * 180 / Math.PI - 90) + ")translate(" + (d.outerRadius - 10) + ")";
  })
  .attr("text-anchor", "end")
  .text(function (d, i) {
    return data[i].label;
  });

// Variable to track if the spin has occurred
var spinOccurred = false;

// Add click event listener
container.on("click", function () {
  // Check if the spin has already occurred
  if (!spinOccurred) {
    // Manually trigger the spin once
    spin();
    // Set the variable to true to prevent further spins
    spinOccurred = true;
  }
});

function spin(d) {
  console.log("OldPick: " + oldpick.length, "Data length: " + data.length);
  if (oldpick.length == data.length) {
    console.log("done");
    return;
  }
  var ps = 360 / data.length,
    pieslice = Math.round(1440 / data.length),
    rng = Math.floor(Math.random() * 1440) + 360;

  rotation = Math.round(rng / ps) * ps;

  picked = Math.round(data.length - (rotation % 360) / ps);
  picked = picked >= data.length ? picked % data.length : picked;
  if (oldpick.indexOf(picked) !== -1 || picked !== 2) {
    d3.select(this).call(spin);
    return;
  } else {
    oldpick.push(picked);
  }
  rotation += 90 - Math.round(ps / 2);
  vis
    .transition()
    .duration(3000)
    .attrTween("transform", rotTween)
    .each("end", function () {
      //populate question
      d3.select("#question h1").text(data[picked].question);
      oldrotation = rotation;

      /* Get the result value from object "data" */
      console.log(data[picked].value);
    });
}

// Make arrow
svg
  .append("g")
  .attr("transform", "translate(" + (w + padding.left + padding.right) + "," + (h / 2 + padding.top) + ")")
  .append("path")
  .attr("d", "M-" + r * 0.15 + ",0L0," + r * 0.05 + "L0,-" + r * 0.05 + "Z")
  .style({ fill: "black" });

// Draw spin circle
container
  .append("circle")
  .attr("cx", 0)
  .attr("cy", 0)
  .attr("r", 60)
  .style({ fill: "white", cursor: "pointer" });

// Spin text
container
  .append("text")
  .attr("x", 0)
  .attr("y", 15)
  .attr("text-anchor", "middle")
  .text("SPIN")
  .style({ "font-weight": "bold", "font-size": "30px" });

function rotTween(to) {
  var i = d3.interpolate(oldrotation % 360, rotation);
  return function (t) {
    return "rotate(" + i(t) + ")";
  };
}
