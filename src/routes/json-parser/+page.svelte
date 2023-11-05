<script lang="ts">
	import Ajv from "ajv";
  
	// Define your JSON schema
	const jsonSchema = {
	  type: "object",
	  properties: {
		// Define your schema properties here
	  },
	  required: [],
	};
  
	let jsonString = ''; // Initial JSON string
	let parsedJson: Record<string, any> | null = null; // Parsed JSON object or null
	let isValidJson = false; // Flag to track JSON validity
	let isSchemaValid = false; // Flag to track JSON schema validity
  
	// Function to update the parsed JSON and check validity
	function updateJson() {
	  try {
		parsedJson = JSON.parse(jsonString);
		isValidJson = true;
  
		// Validate against the JSON schema
		const ajv = new Ajv();
		isSchemaValid = ajv.validate(jsonSchema, parsedJson);
	  } catch (error) {
		parsedJson = null;
		isValidJson = false;
		isSchemaValid = false;
	  }
	}
  </script>
  
  <main>
	<h1>JSON Parser and Validator</h1>
  
	<label for="jsonInput">Enter JSON:</label>
	<textarea
	  id="jsonInput"
	  bind:value={jsonString}
	  on:input={updateJson}
	  placeholder="Enter valid JSON here"
	></textarea>
  
	{#if isValidJson}
	  {#if isSchemaValid}
		<div class="valid">Valid JSON and Schema:</div>
		<pre>{JSON.stringify(parsedJson, null, 2)}</pre>
	  {:else}
		<div class="invalid">Valid JSON but Invalid Schema</div>
	  {/if}
	{:else}
	  <div class="invalid">Invalid JSON</div>
	{/if}
  </main>
  
  <style>
	h1 {
	  text-align: center;
	}
  
	label {
	  display: block;
	  margin-top: 1em;
	}
  
	textarea {
	  width: 100%;
	  height: 200px;
	}
  
	.valid {
	  color: green;
	}
  
	.invalid {
	  color: red;
	}
  </style>
  