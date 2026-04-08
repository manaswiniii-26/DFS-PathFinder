const graph = {
  Arad: ["Zerind", "Sibiu", "Timisoara"], Zerind: ["Arad", "Oradea"],
  Oradea: ["Zerind", "Sibiu"], Sibiu: ["Arad", "Oradea", "Fagaras", "Rimnicu_Vilcea"],
  Timisoara: ["Arad", "Lugoj"], Lugoj: ["Timisoara", "Mehadia"],
  Mehadia: ["Lugoj", "Drobeta"], Drobeta: ["Mehadia", "Craiova"],
  Craiova: ["Drobeta", "Rimnicu_Vilcea", "Pitesti"], Rimnicu_Vilcea: ["Sibiu", "Craiova", "Pitesti"],
  Fagaras: ["Sibiu", "Bucharest"], Pitesti: ["Rimnicu_Vilcea", "Craiova", "Bucharest"],
  Bucharest: ["Fagaras", "Pitesti", "Giurgiu", "Urziceni"], Giurgiu: ["Bucharest"],
  Urziceni: ["Bucharest", "Vaslui", "Hirsova"], Hirsova: ["Urziceni", "Eforie"],
  Eforie: ["Hirsova"], Vaslui: ["Urziceni", "Iasi"],
  Iasi: ["Vaslui", "Neamt"], Neamt: ["Iasi"]
};
 
const coords = {
  Arad: [100, 250], Zerind: [130, 150], Oradea: [200, 80], Sibiu: [300, 280],
  Timisoara: [100, 400], Lugoj: [180, 450], Mehadia: [200, 520], Drobeta: [180, 600],
  Craiova: [320, 600], Rimnicu_Vilcea: [280, 400], Fagaras: [450, 280], Pitesti: [450, 450],
  Bucharest: [600, 500], Giurgiu: [600, 620], Urziceni: [700, 450], Hirsova: [820, 450],
  Eforie: [880, 520], Vaslui: [820, 300], Iasi: [780, 150], Neamt: [650, 100]
};
 
const svg = document.getElementById("map");
const SVG_NS = "http://www.w3.org/2000/svg";
let clickToggle = true;
 
function drawMap() {
  if (!svg) return;
  svg.innerHTML = '';
  for (let city in graph) {
    graph[city].forEach(neigh => {
      const [x1, y1] = coords[city];
      const [x2, y2] = coords[neigh];
      const edge = document.createElementNS(SVG_NS, "line");
      edge.setAttribute("x1", x1); edge.setAttribute("y1", y1);
      edge.setAttribute("x2", x2); edge.setAttribute("y2", y2);
      edge.classList.add("edge");
      edge.id = `edge-${city}-${neigh}`;
      svg.appendChild(edge);
    });
  }
  for (let city in coords) {
    const [x, y] = coords[city];
    const g = document.createElementNS(SVG_NS, "g");
    const circle = document.createElementNS(SVG_NS, "circle");
    circle.setAttribute("cx", x); circle.setAttribute("cy", y);
    circle.setAttribute("r", 14);
    circle.classList.add("node");
    circle.id = city;
    g.onclick = () => {
      if (clickToggle) {
        document.querySelectorAll(".node").forEach(n => n.classList.remove("selected-start"));
        circle.classList.add("selected-start");
        document.getElementById("start").value = city;
      } else {
        document.querySelectorAll(".node").forEach(n => n.classList.remove("selected-goal"));
        circle.classList.add("selected-goal");
        document.getElementById("end").value = city;
      }
      clickToggle = !clickToggle;
    };
    const text = document.createElementNS(SVG_NS, "text");
    text.setAttribute("x", x); text.setAttribute("y", y - 25);
    text.setAttribute("text-anchor", "middle");
    text.classList.add("city-label");
    text.textContent = city;
    g.appendChild(circle);
    g.appendChild(text);
    svg.appendChild(g);
  }
}
 
function getPathDepth(parents, start, goal) {
  let depth = 0, curr = goal;
  while (curr !== start) { curr = parents[curr]; depth++; }
  return depth;
}
 
async function runDFS() {
  const start = document.getElementById("start").value;
  const goal = document.getElementById("end").value;
  resetVisualization();
 
  let stack = [[start, 0]];
  let visited = new Set();
  let parent = {};
  let pathFound = false;
  let maxDepth = 0;
  const log = document.getElementById("visited-log");
 
  // Reset stats
  document.getElementById("stat-max-depth").textContent = "—";
  document.getElementById("stat-path-depth").textContent = "—";
  document.getElementById("stat-nodes").textContent = "—";
 
  while (stack.length > 0) {
    let [curr, depth] = stack[stack.length - 1];
 
    if (depth > maxDepth) {
      maxDepth = depth;
      document.getElementById("stat-max-depth").textContent = maxDepth;
    }
 
    if (!visited.has(curr)) {
      visited.add(curr);
      document.getElementById("stat-nodes").textContent = visited.size;
 
      const el = document.getElementById(curr);
      el.classList.add("current");
      log.innerHTML += `<li>→ <strong>${curr}</strong> <span class="depth-badge">depth ${depth}</span></li>`;
      log.scrollTop = log.scrollHeight;
 
      await new Promise(r => setTimeout(r, 600));
 
      if (curr === goal) {
        pathFound = true;
        const pathDepth = getPathDepth(parent, start, goal);
        document.getElementById("stat-path-depth").textContent = pathDepth;
        document.getElementById("stat-max-depth").textContent = maxDepth;
        highlightPath(parent, start, goal);
        log.innerHTML += `<li class="found-msg">✔ Goal found! Path depth: ${pathDepth} | Max depth: ${maxDepth}</li>`;
        log.scrollTop = log.scrollHeight;
        return;
      }
      el.classList.replace("current", "visited");
    }
 
    let neighbors = graph[curr];
    let next = neighbors.find(n => !visited.has(n));
    if (next) {
      parent[next] = curr;
      stack.push([next, depth + 1]);
    } else {
      let [popped] = stack.pop();
      if (!pathFound) {
        document.getElementById(popped).classList.add("backtrack");
        log.innerHTML += `<li class="backtrack-msg">← Backtrack: ${popped}</li>`;
        log.scrollTop = log.scrollHeight;
        await new Promise(r => setTimeout(r, 300));
      }
    }
  }
 
  document.getElementById("stat-max-depth").textContent = maxDepth;
  log.innerHTML += `<li class="not-found-msg">✘ Goal not reachable!</li>`;
}
 
function highlightPath(parents, start, goal) {
  let curr = goal;
  while (curr !== start) {
    document.getElementById(curr).classList.add("path");
    let prev = parents[curr];
    const edge = document.getElementById(`edge-${prev}-${curr}`) || document.getElementById(`edge-${curr}-${prev}`);
    if (edge) edge.classList.add("path-edge");
    curr = prev;
  }
  document.getElementById(start).classList.add("path");
}
 
function resetVisualization() {
  document.querySelectorAll(".node").forEach(n => n.classList.remove("visited", "current", "backtrack", "path"));
  document.querySelectorAll(".edge").forEach(e => e.classList.remove("path-edge"));
  document.getElementById("visited-log").innerHTML = "";
}
 
document.getElementById("resetBtn").onclick = () => {
  resetVisualization();
  document.querySelectorAll(".node").forEach(n => n.classList.remove("selected-start", "selected-goal"));
  document.getElementById("stat-max-depth").textContent = "—";
  document.getElementById("stat-path-depth").textContent = "—";
  document.getElementById("stat-nodes").textContent = "—";
  clickToggle = true;
};
document.getElementById("startBtn").onclick = runDFS;
 
const s1 = document.getElementById("start");
const s2 = document.getElementById("end");
Object.keys(graph).forEach(c => {
  s1.add(new Option(c, c));
  s2.add(new Option(c, c));
});
 
drawMap();
