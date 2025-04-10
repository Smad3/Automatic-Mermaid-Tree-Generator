<%*
// Constants for parent (tag) prefixes  
// Can be added and modified to include new ones  
const parents = ["t-parent:", "д-батьки:"];
const children = ["t-child:", "д-дитина:"];

const parentsDots = ["t-parent-d:", "д-батьки-т:"];
const childrenDots = ["t-child-d:", "д-дитина-т:"];

// If this text would appear in file/folder name, it won't appear in tree
const progFolder = "t-program-folder";

// Starting of tree creation  
var tree = "\`\`\`mermaid\ngraph TD\n"

// Getting the date/time for checks  
var currentdate = new Date(); 
var dateTime = 
	"Date - Time: " + 
	currentdate.toLocaleDateString(
	"ua-UA", {
	year: "numeric",
	month: "2-digit",
	day: "2-digit",}) + " - " + 
	currentdate.toLocaleTimeString("ua-UA");

// Retrieving paths to all files  
const vault = app.vault;
const allFiles = vault.getMarkdownFiles();

// Active file  
const activeFile = app.workspace.getActiveFile();
const activeFilePath = activeFile.path; 
// Current folder  
const currentFolder = activeFilePath.split('/').slice(0, -1).join('/'); 

// Variable for outputting data  
let output = dateTime + "\n";  
// Variable with all paths in the folder  
let files = [];  

// Filtering files by the current folder  
for (let file of allFiles) {
	let path = file.path;
	if (path.startsWith(currentFolder + "/") && path !== activeFilePath && !path.includes(progFolder)) {
		console.log(path + "\n");
		files.push(path);
	}
}
// Declaring tree elements  
for await (let file of files) {
	let clean = file.slice(file.lastIndexOf("/") + 1, file.lastIndexOf("."));
	tree += clean.replaceAll(" ", "") + "[" + clean + "]\n";
}
tree += "\nclass "
// Adding links in the tree  
for await (let file of files) {
	let clean = file.slice(file.lastIndexOf("/") + 1, file.lastIndexOf("."));
	tree += clean.replaceAll(" ", "") + ",";
}
tree += " internal-link\n"

// Function to find links by tags  
async function findRelatives(path, relative = [""]) {
	let parents = [];
	
    // Retrieving file content  
    const file = app.vault.getAbstractFileByPath(path);
    if (!file) {
        console.error("Файл не знайдено: ", path);
        return parents;
    }
    const content = await app.vault.read(file);
	
  // Regular expression to search for t-relative:[[File Name]]  
  for (let rel of relative) {
		let cont = content
		let pos = content.indexOf(rel);
		for (let i = 0; pos != -1; i++) {
			pos += rel.length;
			cont = cont.slice(pos);
			if(cont.indexOf("]") != -1) {
				if (cont[0] == "[") {
					if (cont[1] == "[" && cont[cont.indexOf("]") + 1] == "]") {
						parents.push(cont.slice(2, cont.indexOf("]")));
					} else {
						parents.push(cont.slice(1, cont.indexOf("]") - 1));
					}
				}
			}
			pos = cont.indexOf(rel);
			if (i >= 500) {console.error("Помилка №500;"); return -1;}
		}
	}
	
	return parents;
}

// Generating connections between blocks  
for (let path of files) {
	let clean = path.slice(path.lastIndexOf("/") + 1, path.lastIndexOf("."));
	for (let parent of await findRelatives(path, parents)) {
		tree += parent.replaceAll(" ", "") + " --> " + clean.replaceAll(" ", "") + "\n";
	}
	for (let child of await findRelatives(path, children)) {
		tree += clean.replaceAll(" ", "") + " --> " + child.replaceAll(" ", "") + "\n";
	}
	for (let parent of await findRelatives(path, parentsDots)) {
		tree += parent.replaceAll(" ", "") + " -.-> " + clean.replaceAll(" ", "") + "\n";
	}
	for (let child of await findRelatives(path, childrenDots)) {
		tree += clean.replaceAll(" ", "") + " -.-> " + child.replaceAll(" ", "") + "\n";
	}
}

tree = Array.from(new Set(tree.split('\n'))).join("\n");

// Closing the diagram  
tree += "\n\`\`\`"
console.log("---------------------------------------");

// Outputting date/time and diagram  
return output + "\n" + tree + "\n";
-%>
