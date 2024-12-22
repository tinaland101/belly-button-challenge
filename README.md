/ Build the metadata panel
function buildMetadata(sample) {
  // Fetch the data from the samples.json file
  d3.json("https://static.bc-edx.com/data/dl-1-2/m14/lms/starter/samples.json").then((data) => {

    // Get the metadata field
    const metadata = data.metadata;

    // Filter the metadata for the object with the desired sample number
    const resultArray = metadata.filter(sampleObj => sampleObj.id == sample);
    const result = resultArray[0];  // There should only be one result since ID is unique

    // Use d3 to select the panel with id of #sample-metadata
    const panel = d3.select("#sample-metadata");

    // Use .html("") to clear any existing metadata
    panel.html("");

    // Loop through the metadata and display each key-value pair with capitalized labels
    Object.entries(result).forEach(([key, value]) => {
      // Capitalize the key (e.g., 'id' to 'ID')
      const formattedKey = key.toUpperCase(); 

      // Append the formatted key and value to the panel
      panel.append("p").text(`${formattedKey}: ${value}`);
    });
  });
}

// Function to build both charts (Bubble and Bar charts)
function buildCharts(sample) {
  d3.json("https://static.bc-edx.com/data/dl-1-2/m14/lms/starter/samples.json").then((data) => {

    // Get the samples field
    const sampleData = data.samples;

    // Filter the samples for the object with the desired sample number
    const resultArray = sampleData.filter(sampleObj => sampleObj.id == sample);
    const result = resultArray[0];  // Get the first result (unique ID)

    // Get the otu_ids, otu_labels, and sample_values
    const otu_ids = result.otu_ids;
    const otu_labels = result.otu_labels;
    const sample_values = result.sample_values;

    // Build the Bubble Chart
    const bubbleTrace = {
      x: otu_ids,
      y: sample_values,
      text: otu_labels,
      mode: 'markers',
      marker: {
        size: sample_values,
        color: otu_ids,
        colorscale: 'Earth'
      }
    };

    const bubbleLayout = {
      title: 'Bacteria Cultures Per Sample',
      xaxis: { title: 'OTU ID' },
      yaxis: {
        title: 'Number of Bacteria',
        tickvals: [0, 500, 1000, 1500, 2000, 2500, 3000, 3500], // Set y-axis ticks
      },
      showlegend: false
    };

    // Render the Bubble Chart
    Plotly.newPlot('bubble', [bubbleTrace], bubbleLayout);

    // For the Bar Chart, map the otu_ids to a list of strings for your yticks
    const top10OTUs = sample_values.slice(0, 10).reverse();
    const top10OTUIds = otu_ids.slice(0, 10).map(id => `OTU ${id}`).reverse();
    const top10OTULabels = otu_labels.slice(0, 10).reverse();

    // Build the Bar Chart
    const barTrace = {
      x: top10OTUs,
      y: top10OTUIds,
      text: top10OTULabels,
      type: 'bar',
      orientation: 'h'
    };

    const barLayout = {
      title: 'Top 10 Bacteria Cultures Found', // Updated title
      xaxis: { 
        title: 'Number of Bacteria', // Updated x-axis label
        tickvals: [0, 20, 40, 60, 80, 100, 120, 140, 160], // Set x-axis ticks
      },
      
    };

    // Render the Bar Chart
    Plotly.newPlot('bar', [barTrace], barLayout);
  });
}

// Function to run on page load
function init() {
  d3.json("https://static.bc-edx.com/data/dl-1-2/m14/lms/starter/samples.json").then((data) => {

    // Get the names field (test subject IDs)
    const sampleNames = data.names;

    // Use d3 to select the dropdown with id of #selDataset
    const dropdown = d3.select("#selDataset");

    // Use the list of sample names to populate the select options
    sampleNames.forEach((sample) => {
      dropdown.append("option").text(sample).property("value", sample);
    });

    // Get the first sample from the list
    const firstSample = sampleNames[0];

    // Build charts and metadata panel with the first sample
    buildCharts(firstSample);
    buildMetadata(firstSample);
  });
}

// Function for event listener when a new sample is selected
function optionChanged(newSample) {
  // Build charts and metadata panel each time a new sample is selected
  buildCharts(newSample);
  buildMetadata(newSample);
}

// Initialize the dashboard
init();
________________
