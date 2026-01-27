<template>
	<div @click="hideContextMenu">
		<section id="container-tools">
			<input type="file" @change="getCsv" />
			<button @click="downloadCsv(dataCsv)">Descargar</button>
			<input type="checkbox" v-model="isTitle" />fila titulo
			<button @click="addColumn">Agregar Columna</button>
			<button @click="addRow">Agregar Fila</button>
      {{ maxColumn }}
		</section>
		<div style="width: 100%; height: 800px; overflow: scroll">
			<!-- Context Menu -->
			<div v-if="contextMenu.visible" :style="{
				position: 'fixed',
				top: contextMenu.y + 'px',
				left: contextMenu.x + 'px',
				backgroundColor: '#fff',
				border: '1px solid #ccc',
				borderRadius: '4px',
				boxShadow: '0 2px 8px rgba(0,0,0,0.15)',
				zIndex: 1000,
				minWidth: '150px'
			}">
				<div v-if="contextMenu.type === 'column'" @click="deleteColumn(contextMenu.index)" style="
					padding: '8px 12px',
					cursor: 'pointer'
				" onmouseover="this.style.backgroundColor='#f0f0f0'" onmouseout="this.style.backgroundColor='#fff'">
					Eliminar Columna
				</div>
				<div v-if="contextMenu.type === 'row'" @click="deleteRow(contextMenu.index)" style="
					padding: '8px 12px',
					cursor: 'pointer'
				" onmouseover="this.style.backgroundColor='#f0f0f0'" onmouseout="this.style.backgroundColor='#fff'">
					Eliminar Fila
				</div>
			</div>
			<table>
				<tr>
					<td></td>
					<td
						name="column-reference"
						@contextmenu.prevent="showContextMenu($event, 'column', index)"
						:style="
							index === active.col
								? ' background-color: gray;  '
								: '  background-color: #E5FFCF; '
						"
						style="position: sticky; top: 0; color: #000"
						v-for="(col, index) in maxColumn"
						:key="index"
					>
						<div :style="'width:' + width + 'px'">
							{{ numToChars(index) }}
						</div>
					</td>
				</tr>
				<tr v-for="(dataRow, i) in dataCsv" :key="i">
					<td
						name="row-reference"
						@contextmenu.prevent="showContextMenu($event, 'row', i)"
						:style="
							i === active.row ? ' background-color: gray;' : '  background-color: #ccc; '
						"
						style="position: sticky; left: 0; color: #000; border: 1px solid #fff"
					>
						{{ i + 1 }}
					</td>
					<td v-for="(dataCol, j) in dataRow" :key="j">
						<input v-model="dataCsv[i][j]" @click="cellActive(i, j)" />
					</td>
				</tr>
			</table>
		</div>
	</div>
</template>

<script>
	import {computed, ref} from 'vue';
	export default {
		setup(props, {emit}) {
			const width = ref(175);
			const heigth = ref(50);
			const active = ref({});
			const contextMenu = ref({
				visible: false,
				x: 0,
				y: 0,
				type: null,
				index: null
			});

			const data = ref('');
			const getCsv = function (e) {
				
				const file = e.target.files[0];
				const reader = new FileReader();
				reader.onload = function (e) {
					data.value = e.target.result;
					emit('update:modelValue', data.value);
          saveDataToLocalStorage(data.value);
				};
        
				reader.readAsText(file);
			};
			const csvToArray = function (str, delimiter = ',') {
				let rows = str.split('\n');
				for (let i = 0; i < rows.length; i++) {
					const element = rows[i].split(delimiter);
					rows[i] = element;
				}
				return rows;
			};

			const numToChars = function (num) {
				numToChars.letters =
					numToChars.letters || 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('');

				if (num < numToChars.letters.length) {
					return numToChars.letters[num];
				} else {
					const first = Math.floor(num / 26) - 1;
					const second = num % 26;
					return numToChars(first) + numToChars.letters[second];
				}
			};
			const getMaxColumn = function (arr) {
				let max = 0;
				if (arr) {
					arr.forEach((elArr) => {
						if (elArr.length > max) {
							max = elArr.length;
						}
					});
				}
				return max;
			};

			const cellActive = function (indexRow, indexCol) {
				console.log('index', indexRow, indexCol);
				active.value = {
					row: indexRow,
					col: indexCol,
				};
			};

			const dataCsv = computed({
				get() {
					return csvToArray(data.value);
				},
				set(value) {
					emit('update:data', value);
				},
			});

			const maxColumn = computed(() => {
				return getMaxColumn(csvToArray(data.value));
			});

			const downloadCsv = (array) => {
				console.log('array', array);
				// (B) ARRAY TO CSV STRING
				let csv = '';
				array.forEach((row) => {
					row.forEach((col, index) => {
						csv += col;
						if (index !== row.length - 1) {
							csv += ',';
						}
					});
					csv += '\n';
				});
				var myBlob = new Blob([csv], {type: 'text/csv'});
				var url = window.URL.createObjectURL(myBlob);
				var anchor = document.createElement('a');
				anchor.href = url;
				anchor.download = 'demo.csv';
				anchor.click();
				window.URL.revokeObjectURL(url);
				anchor.remove();
			};

			const addColumn = () => {
				const newCsv = csvToArray(data.value);
				newCsv.forEach((row) => {
					row.push('');
				});
				const csvString = newCsv.map(row => row.join(',')).join('\n');
				data.value = csvString;
			};

			const addRow = () => {
				const newCsv = csvToArray(data.value);
				const newRow = new Array(maxColumn.value).fill('');
				newCsv.push(newRow);
				const csvString = newCsv.map(row => row.join(',')).join('\n');
				data.value = csvString;
			};

			const saveDataToLocalStorage = (data) => {
				localStorage.setItem('lastCsvData', JSON.stringify(data));
			};

			const loadLastCsvData = () => {
				const lastCsvData = localStorage.getItem('lastCsvData');
				if (lastCsvData) {
					this.data = JSON.parse(lastCsvData);
				}
			};

			const showContextMenu = (event, type, index) => {
				contextMenu.value = {
					visible: true,
					x: event.clientX,
					y: event.clientY,
					type: type,
					index: index
				};
			};

			const hideContextMenu = () => {
				contextMenu.value.visible = false;
			};

			const deleteColumn = (colIndex) => {
				const newCsv = csvToArray(data.value);
				newCsv.forEach((row) => {
					row.splice(colIndex, 1);
				});
				const csvString = newCsv.map(row => row.join(',')).join('\n');
				data.value = csvString;
				hideContextMenu();
			};

			const deleteRow = (rowIndex) => {
				const newCsv = csvToArray(data.value);
				newCsv.splice(rowIndex, 1);
				const csvString = newCsv.map(row => row.join(',')).join('\n');
				data.value = csvString;
				hideContextMenu();
			};

			return {
				dataCsv,
				numToChars,
				maxColumn,
				width,
				heigth,
				active,
				contextMenu,
				cellActive,
				downloadCsv,
				getCsv,
				addColumn,
				addRow,
				loadLastCsvData,
				showContextMenu,
				hideContextMenu,
				deleteColumn,
				deleteRow
			};
		},
	};
</script>

<style>
	#container-tools {
		margin: 20px;
	}
	table {
		border-collapse: collapse;
		width: 100%;
	}
	tr {
		border: 1px solid #ccc;
	}
	td {
		border: 1px solid #ccc;
	}
	td input {
		width: 100%;
		height: 100%;
		margin: 0;
		padding: 0;
		border: 0;
	}
</style>
