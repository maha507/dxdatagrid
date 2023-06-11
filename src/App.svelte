<script>
	import { onMount } from "svelte";
	import "bootstrap/dist/css/bootstrap.min.css";
	import DevExpress from "devextreme";
  
	let jsonData = [];
	let gridData = [];
  
	onMount(async () => {
	  const fetchData = async () => {
		try {
		  const response = await fetch(
			"https://api.recruitly.io/api/candidate?apiKey=TEST9349C0221517DA4942E39B5DF18C68CDA154"
		  );
		  const responseData = await response.json();
		  jsonData = responseData.data;
  
		  gridData = jsonData.map((item) => ({
			name: item.fullName,
			email: item.email,
			phone: item.mobile,
		  }));
  
		  renderDataGrid();
		} catch (error) {
		  console.error("Failed to fetch data:", error);
		}
	  };
  
	  fetchData();
	});
  
	const renderDataGrid = () => {
	  const dataGrid = new DevExpress.ui.dxDataGrid(
		document.getElementById("dataGrid"),
		{
		  dataSource: gridData,
		  columns: [
			{ dataField: "name", caption: "Name", width: 200 },
			{ dataField: "email", caption: "Email", width: 200 },
			{ dataField: "phone", caption: "Mobile", width: 150 },
			// Define other columns as needed
		  ],
		  showBorders: true,
		  filterRow: {
			visible: true,
		  },
		  editing: {
			allowDeleting: true,
			allowAdding: true,
			allowUpdating: true,
			mode: "popup",
			form: {
			  labelLocation: "top",
			},
			popup: {
			  showTitle: true,
			  title: "Row in the editing state",
			},
			texts: {
			  saveRowChanges: "Save",
			  cancelRowChanges: "Cancel",
			  deleteRow: "Delete",
			  confirmDeleteMessage: "Are you sure you want to delete this record?",
			},
		  },
		  paging: {
			pageSize: 20,
		  },
		  onRowInserting: async (e) => {
			console.log("Data being sent to API:", e.data);
			try {
			  const response = await fetch(
				"https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA",
				{
				  method: "POST",
				  headers: {
					"Content-Type": "application/json",
					apiKey: "TEST9349C0221517DA4942E39B5DF18C68CDA154",
				  },
				  body: JSON.stringify(e.data),
				}
			  );
  
			  const responseData = await response.json();
			  if (response.ok) {
				
				e.data.name=responseData.name;
				gridData.push(e.data);
				dataGrid.refresh();
			  } else {
				console.error("Failed to add record:", responseData.error);
			  }
			} catch (error) {
			  console.error("Failed to add record:", error);
			}
		  },
  
		  
		  onRowUpdating: async (e) => {
  console.log("Data sent to API:", e.newData);
  try {
    const updatedData = {
      name: e.newData.name,
      email: e.newData.email,
      phone: e.newData.phone,
    };

    const response = await fetch(
      `https://api.recruitly.io/api/candidate/${e.key}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          apiKey: "TEST9349C0221517DA4942E39B5DF18C68CDA154",
        },
        body: JSON.stringify(e.newData),
      }
    );

    const responseData = await response.json();
    if (response.ok) {
      const updatedItemIndex = gridData.findIndex((item) => item.id === e.key);
      gridData[updatedItemIndex] = e.newData;
      dataGrid.refresh();
    } else {
      console.error("Failed to update record:", responseData.error);
    }
  } catch (error) {
    console.error("Failed to update record:", error);
  }
},

  
		  onRowRemoving: async (e) => {
			console.log("Data being sent to API:", e.data);
			try {
			  const response = await fetch(
				`https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
				{
				  method: "DELETE",
				  headers: {
					"Content-Type": "application/json",
					apiKey: "TEST9349C0221517DA4942E39B5DF18C68CDA154",
				  },
				}
			  );
  
			  if (response.ok) {
				const removedItemIndex = gridData.findIndex((item) => item.id === e.key);
				gridData.splice(removedItemIndex, 1);
				dataGrid.refresh();
			  } else {
				console.error("Failed to delete record.");
			  }
			} catch (error) {
			  console.error("Failed to delete record:", error);
			}
		  },
  
		  onInitialized: () => {
			// Function called when the grid is initialized
			// ...
		  },
		}
	  );
  
	  dataGrid.render();
	};
  </script>
  
  <div id="dataGrid"></div>
  