<%*
// Константи для префіксів (тегів) родичів
// Можна додавати та змінювати, щоб додавати нові
const parents = ["t-parent:", "д-батьки:"];
const children = ["t-child:", "д-дитина:"];

const parentsDots = ["t-parent-d:", "д-батьки-т:"];
const childrenDots = ["t-child-d:", "д-дитина-т:"];

// Голова дерева
var tree = "\`\`\`mermaid\ngraph TD\n"

// Отримання дати/часу для перевірок
var currentdate = new Date(); 
var dateTime = 
	"Date - Time: " + 
	currentdate.toLocaleDateString(
	"ua-UA", {
	year: "numeric",
	month: "2-digit",
	day: "2-digit",}) + " - " + 
	currentdate.toLocaleTimeString("ua-UA");

// Отримання шляхів до усіх файлів
const vault = app.vault;
const allFiles = vault.getMarkdownFiles();

// Активний файл
const activeFile = app.workspace.getActiveFile();
const activeFilePath = activeFile.path; 
// Поточна тека
const currentFolder = activeFilePath.split('/').slice(0, -1).join('/'); 

// Змінна для виводу даних
let output = dateTime + "\n";
// змінна з усіма шляхами в папці
let files = [];

// Фільтр файлів до поточної теки
for (let file of allFiles) {
	let path = file.path;
	if (path.startsWith(currentFolder + "/") && path !== activeFilePath) {
		console.log(path + "\n");
		files.push(path);
	}
}
// Об'явлення елементів дерева
for await (let file of files) {
	let clean = file.slice(file.lastIndexOf("/") + 1, file.lastIndexOf("."));
	tree += clean.replaceAll(" ", "") + "[" + clean + "]\n";
}
tree += "\nclass "
// Додавання посилань у дереві
for await (let file of files) {
	let clean = file.slice(file.lastIndexOf("/") + 1, file.lastIndexOf("."));
	tree += clean.replaceAll(" ", "") + ",";
}
tree += " internal-link\n"

// Функція для знаходження посилань за теґами
async function findRelatives(path, relative = [""]) {
	let parents = [];
	
	// Отримання вмісту файлу
    const file = app.vault.getAbstractFileByPath(path);
    if (!file) {
        console.error("Файл не знайдено: ", path);
        return parents;
    }
    const content = await app.vault.read(file);
	
    // Регулярний вираз для пошуку t-relative:[[File Name]]
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

//Генерація зв'язків між блоками
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

// Закривання діаграми
tree += "\n\`\`\`"
console.log("---------------------------------------");

// Вивід дати/часу та діаграми
return output + "\n" + tree + "\n";
-%>
