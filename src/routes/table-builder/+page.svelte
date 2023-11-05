<script lang="ts">
    import Ajv from "ajv";
    import { onMount } from "svelte";
      
    // Define your JSON schema
    const jsonSchema = {
        type: "object",
        properties: {
            section: { type: "string" },
            section_num: { type: "number" },
            table_data: {
                type: "array",
                items: {
                    type: "object",
                    properties: {
                        col_header: { type: "boolean" },
                        row_header: { type: "boolean" },
                        sub_header: { type: "boolean" },
                        total_row:  { type: "boolean" },
                        disabled:   { type: "boolean" },
                        bold:       { type: "boolean" },
                        small:      { type: "boolean" },
                        can_link:   { type: "boolean" },
                        value:  { type: "string" },
                        field:  { type: "string" },
                        width:  { type: "string" },
                        align:  { type: "string" },
                        format: { type: "string" },
                    },
                    required: [],
                },
            },
        },
        required: ["section", "section_num", "table_data"],
    };
  
    let jsonString = ''; // Initial JSON string
    let parsedJson: Record<string, any> | null = null; // Parsed JSON object or null
    let isValidJson = false; // Flag to track JSON validity
    let isSchemaValid = false; // Flag to track JSON schema validity

    // State for handling cell menu
    let isCellMenuOpen = false;
    let selectedCell: Record<string, any> = {}; // The selected cell's data
    let editedValues: Record<string, any> = {};
  
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

    // Function to open cell menu
    function openCellMenu(cell: { value: any; }) {
        isCellMenuOpen = true;
        selectedCell = cell;
        editedValues = { ...cell };
    }

    // Function to close cell menu
    function closeCellMenu() {
        isCellMenuOpen = false;
        selectedCell = {};
    }

    // Function to save edited value
    function saveEditedValues() {
        if (selectedCell) {
              // Update the selectedCell attributes with editedValue properties
            for (const key in editedValues) {
                if (Object.prototype.hasOwnProperty.call(editedValues, key)) {
                    selectedCell[key] = editedValues[key];
                }
            }
            updateJsonString();
            closeCellMenu();
        }
    }

    // Function to copy the JSON array portion to the clipboard
    function copyJsonArrayToClipboard() {
        if (parsedJson && parsedJson.table_data) {
            // Convert the data array portion to a JSON string
            const jsonArrayString = JSON.stringify(parsedJson.table_data);

            // Use the Clipboard API to copy the JSON array to the clipboard
            navigator.clipboard.writeText(jsonArrayString).then(() => {
                alert("Table Data copied to clipboard!");
            }).catch((error) => {
                console.error("Error copying to clipboard:", error);
            });
            } else {
            alert("No JSON table data to copy.");
        }
    }

	// Get the number of columns
	function getColSize() {
		let size = 1;
		if (parsedJson) {
			size = parsedJson.table_data.filter((c: { col_header: any; }) => c.col_header).length;
			size = (size < 1) ? parsedJson.table_data.length : size;
		}

		return (size < 1) ? 1 : size;
	}

    // Function to update the JSON string and validate
    function updateJsonString() {
        if (parsedJson) {
            jsonString = JSON.stringify(parsedJson);

            // Validate against the JSON schema
            const ajv = new Ajv();
            isSchemaValid = ajv.validate(jsonSchema, parsedJson);
        } else {
            jsonString = ''; // Set jsonString to null when parsedJson is null
            isSchemaValid = false;
        }
        updateJson();
    }

    // Handle initial JSON parsing and validation
    onMount(() => {
        updateJsonString();
    });
  </script>
  
  <main>
    <h1>JSON Validator and Table Editor</h1>
  
    <label for="jsonInput">Enter JSON:</label>
    <textarea
      id="jsonInput"
      bind:value={jsonString}
      on:input={updateJson}
      placeholder="Enter valid JSON here"
    ></textarea>
  
    {#if isValidJson}
      {#if isSchemaValid}
        <div class="valid">Valid JSON and Schema: 
            <button on:click={copyJsonArrayToClipboard}>Copy to Clipboard</button>
        </div>
      {:else}
        <div class="invalid">Valid JSON but Invalid Schema</div>
      {/if}
    {:else}
      <div class="invalid">Invalid JSON</div>
    {/if}
	<br />
  </main>
  
  {#if isSchemaValid && parsedJson && parsedJson.table_data && parsedJson.table_data.length > 0}
    <section>
        <!-- Create the table from table_data -->
        <table>
            <thead>
                <tr>
                    <th colspan={getColSize()}>{parsedJson.section}</th>
                </tr>
            </thead>
            <tbody>
                {#each parsedJson.table_data as item, colIndex}
                    {#if item.row_header || item.sub_header}
                        <tr>
                    {/if}

                    {#if item.disabled}
                        <td class="disabled"></td>
                    {:else}
                        <td
                            on:click={() => openCellMenu(item)}
                            class="editable"
                            class:col-header-1st={item.col_header && colIndex == 0}
                            class:col-header={(item.col_header && colIndex > 0) || item.total_row}
                            class:row-header={item.row_header}
                            class:field-linked={item.field && item.can_link}
                            class:field-no-link={item.field && !item.can_link}
                            class:bold-font={item.bold}
                            class:small-font={item.small}
                        >
                            {item.value || item.field || ''} {#if item.format}(%){/if}
                        </td>
                    {/if}
                {/each}
            </tbody>
        </table>
    </section>
  {:else if isSchemaValid && parsedJson && (!parsedJson.data || parsedJson.data.length === 0)}
    <section>
        <p>No data to display.</p>
    </section>
  {/if}

  {#if isCellMenuOpen}
    <div class="cell-menu" class:open>
        <div class="cell-menu-content">
            <h3>Edit Cell</h3>
            <div class="form-field">
                <label for="editedValue.col_header">Col Header:</label>
                <input type="checkbox" id="editedValue.col_header" bind:checked={editedValues.col_header} />
            </div>
            <div class="form-field">
                <label for="editedValue.row_header">Row Header:</label>
                <input type="checkbox" id="editedValue.row_header" bind:checked={editedValues.row_header} />
            </div>
            <div class="form-field">
                <label for="editedValue.sub_header">Sub Header:</label>
                <input type="checkbox" id="editedValue.sub_header" bind:checked={editedValues.sub_header} />
            </div>
            <div class="form-field">
                <label for="editedValue.total_row">Total Row:</label>
                <input type="checkbox" id="editedValue.total_row" bind:checked={editedValues.total_row} />
            </div>
            <div class="form-field">
                <label for="editedValue.disabled">Disabled:</label>
                <input type="checkbox" id="editedValue.disabled" bind:checked={editedValues.disabled} />
            </div>
            <div class="form-field">
                <label for="editedValue.bold">Bold:</label>
                <input type="checkbox" id="editedValue.bold" bind:checked={editedValues.bold} />
            </div>
            <div class="form-field">
                <label for="editedValue.small">Small Font:</label>
                <input type="checkbox" id="editedValue.small" bind:checked={editedValues.small} />
            </div>
            <div class="form-field">
                <label for="editedValue.can_link">Can Link:</label>
                <input type="checkbox" id="editedValue.can_link" bind:checked={editedValues.can_link} />
            </div>
            <label for="editedValue.width">Width (%):</label>
            <input type="text" id="editedValue.width" bind:value={editedValues.width} />
            <label for="editedValue.align">Alignment:</label>
            <input type="text" id="editedValue.align" bind:value={editedValues.align} />
            <label for="editedValue.format">Render Format:</label>
            <input type="text" id="editedValue.format" bind:value={editedValues.format} />
            <label for="editedValue.value">Value:</label>
            <input type="text" id="editedValue.value" bind:value={editedValues.value} />
            <label for="editedValue.field">Field:</label>
            <input type="text" id="editedValue.field" bind:value={editedValues.field} />
            <hr />    
            <div class="button-container">
                <button on:click={saveEditedValues}>Save</button>
                <button on:click={closeCellMenu}>Cancel</button>
            </div>
        </div>
    </div>
  {/if}

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

    /* Table styling */
    table {
        width: 100%;
        border-collapse: collapse;
        border: solid 1px;
    }

    td {
        border: 1px solid #dddddd;
        text-align: left;
        padding: 8px;
    }

    th {
        border: 1px solid #dddddd;
        text-align: left;
        background-color: #8f8f8f;
        padding: 8px;
    }

    tr:nth-child(even) {
        background-color: #f2f2f2;
    }

    /* Additional styling for editable cells */
    td.editable {
        cursor: pointer;
    }


	/* Attribute Styling */
    section {
        display: flex;
        justify-content: center;
    }

	.col-header-1st {
		font-weight: bold;
		text-align: left;
        background-color: #c4c2c2;
	}

	.col-header {
		font-weight: bold;
		text-align: center;
        background-color: #c4c2c2;
	}

	.row-header {
		font-weight: bold;
		text-align: left;
	}

	.field-linked {
		font-weight: bold;
		text-align: center;
		color: blue;
	}

	.field-no-link {
		font-weight: bold;
		text-align: center;
	}

	.disabled {
		background-color: #c4c2c2;
	}

	.bold-font {
		font-weight: bold;
	}

	.small-font {
		font-size: small;
	}

    /* Cell Menu styling */
    .cell-menu {
        position: fixed;
        top: 0;
        right: -300px; /* Start off-screen to the right */
        width: 300px;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        z-index: 999;
        transition: right 0.3s ease-in-out; /* Slide-in transition */
    }

    .cell-menu-content {
        background-color: white;
        padding: 20px;
        border-radius: 4px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    }

    .cell-menu.open {
        right: 0;
    }

    .form-field {
        display: block;
        margin-bottom: 10px;
    }

    .form-field label {
        display: inline-block;
        vertical-align: middle;
        margin-right: 10px;
    }

    .form-field input[type="checkbox"] {
        display: inline-block;
        vertical-align: bottom;
    }

    .button-container {
        display: flex;
        justify-content: center;
    }
  </style>
  