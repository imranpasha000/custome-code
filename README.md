import { Checkbox } from "@/components/ui/checkbox";
import { ChevronDown, ChevronRight } from "lucide-react";
import React, { ReactNode, useEffect, useState } from "react";


interface TreeNode {
label: ReactNode;
id: number;
code: string;
ics: string;
children?: TreeNode[];
}

const IcsClassification: React.FC = () => {
const [data, setData] = useState<TreeNode[]>([]);
const [expandedNodes, setExpandedNodes] = useState<Set<number>>(new Set());

const [selectedNodes, setSelectedNodes] = useState<Set<number>>(new Set());

useEffect(() => {
const fetchData = async () => {
const response = await fetch("/api/v1/ics");
const result: any[] = await response.json();

const buildTree = (items: any[]): TreeNode[] => {
return items.map((item) => ({
id: item.id,
label: `${item.code} - ${item.ics}`,
code: item.code,
ics: item.ics,
children: item.subIcsClassifications
? buildTree(item.subIcsClassifications)
: [],
}));
};

setData(buildTree(result));
};

fetchData();
}, []);
console.log(selectedNodes);

const handleCheckboxChange = (id: number, isSelected: boolean = false) => {
const newSelectedNodes = new Set(selectedNodes);

const traverse = (node: TreeNode, select: boolean) => {
if (select) {
newSelectedNodes.add(node.id);
} else {
newSelectedNodes.delete(node.id);
}

node.children?.forEach((child) => traverse(child, select));
};

const unselectParent = (data: TreeNode, id: number) => {
if (data.children == null) return false;
for (let i of data.children) {
if (i.id == id) {
//remove myself
newSelectedNodes.delete(i.id);
//remove parent
newSelectedNodes.delete(data.id);
return true;
}
let flag = unselectParent(i, id);
if (flag) {
newSelectedNodes.delete(data.id);
return true;
}
}
};

const node = findNode(data, id);
if (node) {
traverse(node, isSelected ?? false);

if (!isSelected) {
for (let i of data) {
let flag = unselectParent(i, id);
if (flag) break;
}
}
}

setSelectedNodes(newSelectedNodes);
};

const findNode = (data: TreeNode[], id: number): TreeNode | null => {
for (const i of data) {
if (i.id === id) return i;
if (i.children && i.children.length > 0) {
const val = findNode(i.children, id);
if (val) return val;
}
}
return null;
};

const handleExpandToggle = (id: number) => {
setExpandedNodes((prev) =>
prev.has(id)
? new Set([...prev].filter((nodeId) => nodeId !== id))
: new Set([...prev, id]),
);
};
const renderTreeNodes = (nodes: TreeNode[]) => {
return nodes.map((node) => (
<div key={node.id} className="ml-4">
<div className="flex items-center space-x-2">
{node.children && node.children.length > 0 && (
<button
className="p-1 rounded hover:bg-gray-200"
onClick={() => handleExpandToggle(node.id)}
>
{expandedNodes.has(node.id) ? (
<ChevronDown size={16} />
) : (
<ChevronRight size={16} />
)}
</button>
)}
<Checkbox
checked={selectedNodes.has(node.id)}
onCheckedChange={(checked) =>
handleCheckboxChange(node.id, checked === true)
}
/>
<span className="text-sm font-medium text-gray-700">
{node.label}
</span>
</div>
{expandedNodes.has(node.id) &&
node.children &&
node.children.length > 0 && (
<div className="ml-6 border-l pl-2">
{renderTreeNodes(node.children)}
</div>
)}
</div>
));
};

return (
<div className="w-64 bg-white border border-gray-300 rounded-lg shadow-lg p-4">
{renderTreeNodes(data)}
</div>
);
};

export default IcsClassification;
// import React, { useState, useEffect } from "react";
// import { toast } from "sonner";
// import { searchStandards, fetchIcsClassifications, fetchSdos, fetchCurrencies } from "@/services/classifiaction/ics_classification";
// import { DataGrid, DataGridColumnHeader } from "@/components";
// import { PanelRightOpen } from "lucide-react";

// interface Sdo {
// id: number;
// title: string;
// }

// interface SubClassification {
// id: number;
// code: string;
// ics: string;
// }

// interface Classification {
// id: number;
// code: string;
// ics: string;
// subIcsClassifications: SubClassification[];
// }

// interface Standard {
// display_stdNo: string;
// stdPrices: {
// formate_id: number;
// price: number;
// }[];
// priceInINR?: number;
// }

// interface Currency {
// id: number;
// currency_amount: number;
// }

// const IcsClassification: React.FC = () => {
// const [sdos, setSdos] = useState<Sdo[]>([]);
// const [selectedSdo, setSelectedSdo] = useState<number | null>(null);
// const [data, setData] = useState<Standard[]>([]);
// const [currencyData, setCurrencyData] = useState<Currency[]>([]);
// const [loading, setLoading] = useState(false);




// // Function to handle search
// const handleSearch = async (selectedIcsIds: number[]) => {
// if (selectedIcsIds.length > 0) {
// setLoading(true);
// try {
// const allSelectedIcsIds = selectedIcsIds.map((id) => Number(id));

// const result = await searchStandards(
// selectedSdo ? [selectedSdo] : [],
// allSelectedIcsIds
// );

// setData(result.data); // Assuming result.data contains an array of `Standard`
// toast.success("Data loaded successfully!");
// } catch (error) {
// toast.error("Failed to load standards");
// } finally {
// setLoading(false);
// }
// } else {
// toast.error("Please select ICS Classification(s)");
// }
// };

// // Fetching initial data
// useEffect(() => {
// const loadInitialData = async () => {
// try {
// const [sdosData, icsData, currencyResponse] = await Promise.all([fetchSdos(), fetchIcsClassifications(), fetchCurrencies()]);
// setSdos(sdosData);
// setIcsClassifications(icsData);
// if (!currencyResponse.error) {
// setCurrencyData(currencyResponse);
// }
// } catch (error) {
// console.error("Error fetching initial data:", error);
// toast.error("Failed to load initial data");
// }
// };
// loadInitialData();
// }, []);

// const columns = [
// {
// accessorFn: (row: Standard) => row.display_stdNo,
// id: "display_stdNo",
// meta: { headerClassName: "bg-gray-500 text-white" },
// header: ({ column }: { column: any }) => (
// <DataGridColumnHeader title="Standard No." column={column} />
// ),
// cell: ({ row }: { row: { original: Standard } }) => (
// <span>{row.original.display_stdNo}</span>
// ),
// },
// {
// accessorFn: (row: Standard) => row.stdPrices,
// id: "stdPrices",
// meta: { headerClassName: "bg-gray-500 text-white" },
// header: ({ column }: { column: any }) => (
// <DataGridColumnHeader title="Unit Rate" column={column} />
// ),
// cell: ({ row }: { row: { original: Standard } }) => (
// <div>
// {row.original.stdPrices?.length > 0 ? (
// row.original.stdPrices.map((priceList, index) => (
// <div key={index} className="inline-block mr-4">
// <p className="inline">
// {priceList.price}
// {priceList.formate_id === 1
// ? " PDF"
// : priceList.formate_id === 2
// ? " PRINT"
// : " Unknown"}
// </p>
// </div>
// ))
// ) : (
// <span>N/A</span>
// )}
// </div>
// ),
// },
// {
// accessorFn: (row: Standard) => row.priceInINR,
// id: "priceInINR",
// meta: { headerClassName: "bg-gray-500 text-white" },
// header: ({ column }: { column: any }) => (
// <DataGridColumnHeader title="Price (INR)" column={column} />
// ),
// cell: ({ row }: { row: { original: Standard } }) => {
// const curr = currencyData.find((c) => c.id === 4);
// const price =
// row.original.stdPrices?.length > 0
// ? row.original.stdPrices[0].price * (curr?.currency_amount || 1)
// : "N/A";
// return <span>{price}</span>;
// },
// },
// {
// id: "actions",
// meta: { headerClassName: "bg-gray-500 text-white" },
// cell: ({ row }: { row: { original: Standard } }) => (
// <button className="bg-blue-500 rounded hover:bg-gray-100 items-center p-2">
// <PanelRightOpen size={16} strokeWidth={3} />
// </button>
// ),
// },
// ];



// return (
// <div className="flex flex-col min-h-screen overflow-x-hidden">
// <nav className="bg-blue-500 p-4 rounded">
// <div className="flex space-x-4">
// {sdos.length > 0 && (
// <select
// value={selectedSdo || ""}
// onChange={(e) => setSelectedSdo(Number(e.target.value))}
// className="border rounded px-4 py-2 w-64 max-w-full text-sm"
// >
// <option value="">All SDOs</option>
// {sdos.map((sdo) => (
// <option key={sdo.id} value={sdo.id} className="text-xs">
// {sdo.title}
// </option>
// ))}
// </select>
// )}
// {/* Search Button */}
// <button
// onClick={handleSearch}
// className="bg-white text-black p-2 rounded hover:bg-gray-500 hover:text-white transition-all duration-300"
// >
// Search
// </button>
// </div>
// </nav>

// <div className="flex">


// {/* Data Grid */}
// <div className="flex-1">
// <div className="p-2">
// {loading ? (
// <div className="flex justify-center items-center h-full space-x-4">
// <div className="w-6 h-6 border-4 border-t-transparent border-blue-500 border-solid rounded-full animate-spin"></div>
// <span className="text-lg">Loading...</span>
// </div>
// ) : (
// <DataGrid
// columns={columns}
// data={data}
// pagination={{ size: 10 }}
// />
// )}
// </div>
// </div>
// </div>
// </div>
// );
// };

// export default IcsClassification;
// {
/*

import React, { useEffect, useState } from "react";
import { toast } from "sonner";
import { DataGrid, DataGridColumnHeader } from "@/components";
import { PanelRightOpen } from "lucide-react";
import { fetchSdos,fetchCurrencies, fetchIcsClassifications,searchStandards } from "@/services/classifiaction/ics_classification";


interface Sdo {
id: number;
title: string;
}

interface SubClassification {
id: number;
code: string;
ics: string;
}

interface Classification {
id: number;
code: string;
ics: string;
subIcsClassifications: SubClassification[];
}


interface Standard {
stdPrice: any;
priceInINR: any;
display_stdNo: string;
stdPrices: {
formate_id: number;
price: number;
lang_id: number;
}[];
classificationId: number;
}

interface Currency {
id: number;
currency_amount: number;
}

const IcsClassification: React.FC = () => {
const [sdos, setSdos] = useState<Sdo[]>([]);
const [icsClassifications, setIcsClassifications] = useState<Classification[]>([]);
const [subIcsClassifications, setSubIcsClassifications] = useState<SubClassification[]>([]);
const [selectedSdo, setSelectedSdo] = useState<number | null>(null);
const [selectedIcsClassification, setSelectedIcsClassification] = useState<number | null>(null);
const [selectedSubIcsClassification, setSelectedSubIcsClassification] = useState<number | null>(null);
const [data, setData] = useState<Standard[]>([]);
console.log(data);
const [currencyData, setCurrencyData] = useState<Currency[]>([]);
const [loading, setLoading] = useState(false);


useEffect(() => {
const loadInitialData = async () => {
try {
const [sdosData, icsData, currencyResponse] = await Promise.all([
fetchSdos(),
fetchIcsClassifications(),
fetchCurrencies(),
]);
setSdos(sdosData);
setIcsClassifications(icsData);
if (!currencyResponse.error) {
setCurrencyData(currencyResponse);
}
} catch (error) {
console.error("Error fetching initial data:", error);
toast.error("Failed to load initial data");
}
};
loadInitialData();
}, []);

// Handle ICS Classification Change (select)
const handleIcsClassificationChange = (classId: number) => {
const selectedClassification = icsClassifications.find((ics) => ics.id === classId);
if (selectedClassification) {
setSubIcsClassifications(selectedClassification.subIcsClassifications);
setSelectedIcsClassification(classId);
setSelectedSubIcsClassification(null); // Reset sub-ICS selection
}
};

// Handle Sub-ICS Classification Change (select)
const handleSubIcsClassificationChange = (subClassId: number) => {
setSelectedSubIcsClassification(subClassId);
};

const handleSearch = async () => {
if (selectedIcsClassification && selectedSubIcsClassification) {
setLoading(true);
try {
const result = await searchStandards(
selectedSdo ? [selectedSdo] : [],
[selectedIcsClassification, selectedSubIcsClassification]
);
setData(result.data);
toast.success("Data loaded successfully!");
} catch (error) {
toast.error("Failed to load standards");
} finally {
setLoading(false);
}
} else {
toast.error("Please select ICS Classification and Sub-ICS Classification");
}
};




const columns = [
{
accessorFn: (row: Standard) => row.display_stdNo,
id: "display_stdNo",
meta: {
headerClassName: "bg-gray-500 text-white",
},
header: ({ column }: { column: any }) => (
<DataGridColumnHeader title="Standard No." column={column} />
),
cell: ({ row }: { row: { original: Standard } }) => (
<span>{row.original.display_stdNo}</span>
),
},
{
accessorFn: (row: Standard) => row.stdPrices,
id: "stdPrices",
meta: {
headerClassName: "bg-gray-500 text-white",
},
header: ({ column }: { column: any }) => (
<DataGridColumnHeader title="Unit Rate" column={column} />
),
cell: ({ row }: { row: { original: Standard } }) => {
return (
<div>
{row.original.stdPrices.map((priceList, index) => (
<div key={index} className="inline-block mr-4">
<p className="inline">
{priceList.price}
{priceList.formate_id === 1
? " PDF"
: priceList.formate_id === 2
? " PRINT"
: " Unknown"}
</p>
</div>
))}
</div>
);
},
},
{
accessorFn: (row: Standard) => row.priceInINR,
id: "priceInINR",
meta: {
headerClassName: "bg-gray-500 text-white",
},
header: ({ column }: { column: any }) => (
<DataGridColumnHeader title="Price (INR)" column={column} />
),
cell: ({ row }: { row: { original: Standard } }) => {
const curr = currencyData.find((c) => c.id === 4);
const price =
row.original.stdPrices && row.original.stdPrices.length > 0
? row.original.stdPrices[0].price * curr.currency_amount
: "N/A";
return <span>{price}</span>;
},
},
{
id: "actions",
meta: {
headerClassName: "bg-gray-500 text-white",
},
cell: ({ row }: { row: { original: Standard } }) => (
<button className="bg-blue rounded hover:bg-gray-100 items-center">
<PanelRightOpen size={16} strokeWidth={3} />
</button>
),
},
];

return (
<div className="flex flex-col min-h-screen overflow-x-hidden">
<nav className="bg-blue-500 p-4 rounded">
<div className="flex space-x-4">

{sdos.length > 0 && (
<select
value={selectedSdo || ""}
onChange={(e) => setSelectedSdo(Number(e.target.value))}
className="border rounded px-4 py-2 w-64 max-w-full text-sm"
>
<option value="">All SDOs</option>
{sdos.map((sdo) => (
<option key={sdo.id} value={sdo.id} className="text-xs">
{sdo.title}
</option>
))}
</select>
)}

{icsClassifications.length > 0 && (
<select
value={selectedIcsClassification || ""}
onChange={(e) => handleIcsClassificationChange(Number(e.target.value))}
className="border rounded px-4 py-2 w-64 max-w-full text-sm"
>
{icsClassifications.map((ics) => (
<option key={ics.id} value={ics.id} className="text-xs">
{`${ics.code} - ${ics.ics}`}
</option>
))}
</select>
)}


{subIcsClassifications.length > 0 && (
<select
value={selectedSubIcsClassification || ""}
onChange={(e) => handleSubIcsClassificationChange(Number(e.target.value))}
className="border rounded px-4 py-2 w-64 max-w-full text-sm"
>
{subIcsClassifications.map((subIcs) => (
<option key={subIcs.id} value={subIcs.id} className="text-xs">
{`${subIcs.code} - ${subIcs.ics}`}
</option>
))}
</select>
)}


<button
onClick={handleSearch}
className="bg-white text-black p-2 rounded hover:bg-gray-500 hover:text-white transition-all duration-300"
>
Search
</button>

</div>
</nav>


<div className="p-4">
{loading ? (
<div className="flex justify-center items-center h-full space-x-4">
<div className="w-6 h-6 border-4 border-t-transparent border-blue-500 border-solid rounded-full animate-spin"></div>
<span className="text-lg">Loading...</span>
</div>
) : (
<DataGrid columns={columns} data={data} pagination={{ size: 10 }} />
)}
</div>
</div>
);
};

export default IcsClassification;
*/
