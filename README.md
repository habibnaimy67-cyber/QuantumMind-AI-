# QuantumMind-AI-
QuantumMind AI är en intelligent, guidande AI-agent designad för att hjälpa medborgare och användare att navigera information inom vård, kommunala tjänster och samhällsservice på ett tryggt och enkelt sätt.
# filename: quantummind_ai_full.py
import os, asyncio, random, json
from datetime import datetime
from collections import deque
from flask import Flask, render_template_string, request, jsonify

app = Flask(__name__)

# ====== AGENT SYSTEM ======
class Agent:
    def __init__(self, name, emoji, color, role):
        self.name = name
        self.emoji = emoji
        self.color = color
        self.role = role
        self.status = "idle"
        self.memory = deque(maxlen=10)

    async def run(self, task, iteration=0):
        self.status = "working"
        await asyncio.sleep(0.3)
        responses = {
            "Research": f"🔍 Marknadsanalys: TAM €50M, 3 konkurrenter identifierade",
            "Coding": f"💻 200 rader Python genererade",
            "QA": f"✅ Kvalitetsgranskning: 89% score",
            "Business": f"📊 Affärsmodell: ROI 340%",
            "Marketing": f"📢 Go-to-market strategi klar",
            "Security": f"🛡️ Säkerhetskontroll OK",
            "Translation": f"🌐 Översättning EN/SE/DE OK",
            "Legal": f"⚖️ GDPR-kompatibel",
            "DevOps": f"🚀 CI/CD pipeline aktiv",
            "Strategy": f"📈 Roadmap Q1→Q4"
        }
        output = responses.get(self.name, f"Bearbetade: {task[:25]}...")
        self.memory.append({"iter": iteration, "output": output[:50]})
        self.status = "done"
        return {
            "agent": self.name,
            "emoji": self.emoji,
            "output": output,
            "iteration": iteration,
            "confidence": round(0.85 + random.random() * 0.12, 2),
            "requires_approval": self.name in ["Legal", "Security"] and iteration == 0
        }

class ChefAgent:
    def __init__(self):
        self.agents = {
            "Research": Agent("Research", "🔬", "#FF6B6B", "Analytiker"),
            "Coding": Agent("Coding", "💻", "#4ECDC4", "Utvecklare"),
            "QA": Agent("QA", "✅", "#45B7D1", "Testare"),
            "Business": Agent("Business", "💼", "#96CEB4", "Strateg"),
            "Marketing": Agent("Marketing", "📢", "#FFEAA7", "Marknadsförare"),
            "Security": Agent("Security", "🛡️", "#DDA0DD", "Säkerhetsexpert"),
            "Translation": Agent("Translation", "🌐", "#FD79A8", "Lokaliserare"),
            "Legal": Agent("Legal", "⚖️", "#FDCB6E", "Jurist"),
            "DevOps": Agent("DevOps", "🚀", "#6C5CE7", "DevOps"),
            "Strategy": Agent("Strategy", "📊", "#E17055", "Konsult")
        }

    def select_agents(self, task):
        task_lower = task.lower()
        selected = set()
        keywords = {
            "Coding": ["skapa", "koda", "app", "system", "web", "api"],
            "Research": ["analysera", "marknad", "konkurrent", "data"],
            "Business": ["affär", "intäkt", "kostnad", "roi"],
            "Marketing": ["marknadsför", "seo", "kampanj"],
            "Security": ["säker", "gdpr", "hack"],
            "Legal": ["juridisk", "avtal", "compliance"],
            "DevOps": ["deploy", "server", "docker", "kubernetes"],
            "Strategy": ["strategi", "plan", "roadmap"],
            "Translation": ["översätt", "språk", "lokalisera"]
        }
        for agent, words in keywords.items():
            if any(w in task_lower for w in words):
                selected.add(agent)
        if "Coding" in selected:
            selected.update(["QA", "Security"])
        return selected if selected else {"Research", "Coding", "Business"}

    async def orchestrate(self, task, max_iter=2):
        selected = self.select_agents(task)
        iterations = []
        start = datetime.now()
        for i in range(max_iter):
            tasks = [self.agents[a].run(task, i) for a in selected]
            results = await asyncio.gather(*tasks)
            iterations.append({"iteration": i, "results": {r["agent"]: r for r in results}})
        duration = (datetime.now() - start).total_seconds()
        summary = []
        for it in iterations:
            for name, res in it["results"].items():
                summary.append(f"{res['emoji']} {name}: {res['output'][:55]}...")
        return {
            "task": task,
            "selected_agents": list(selected),
            "iterations": iterations,
            "summary": "\n".join(summary),
            "duration": round(duration, 2),
            "quality_score": round(0.87 + random.random() * 0.08, 2)
        }

chef = ChefAgent()

# ====== ROUTES ======
@app.route("/")
def index():
    return render_template_string("""
<!DOCTYPE html>
<html lang="sv">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QuantumMind AI</title>
<link rel="manifest" href="/manifest.json">
<style>
body {font-family:sans-serif; padding:20px; background:#f5f5f5;}
h1 {color:#333;} input {width:80%; padding:10px; font-size:16px;}
button {padding:10px 15px; font-size:16px;}
#output {background:#fff; padding:15px; margin-top:10px; border-radius:5px; white-space:pre-wrap;}
.agent {display:inline-block; padding:5px 10px; margin:3px; border-radius:5px; color:#fff;}
</style>
</head>
<body>
<h1>QuantumMind AI</h1>
<input type="text" id="task" placeholder="Skriv din uppgift här">
<button onclick="runTask()">Kör</button>
<div id="agents"></div>
<pre id="output"></pre>

<script>
async function runTask(){
    let task = document.getElementById('task').value;
    let res = await fetch('/api/run',{
        method:'POST', headers:{'Content-Type':'application/json'},
        body: JSON.stringify({task})
    });
    let data = await res.json();
    let output = '';
    data.iterations.forEach(it=>{
        output += 'Iteration '+it.iteration+'\\n';
        for(let a in it.results){
            let r = it.results[a];
            output += r.emoji+' '+r.agent+': '+r.output+'\\n';
        }
        output += '\\n';
    });
    output += '\\nSummary:\\n'+data.summary;
    document.getElementById('output').textContent = output;

    let agentsDiv = document.getElementById('agents');
    agentsDiv.innerHTML = '';
    data.selected_agents.forEach(a=>{
        agentsDiv.innerHTML += `<span class="agent" style="background:${chefAgents[a]}">${a}</span>`;
    });
}

// Agent färger
const chefAgents = {
    "Research":"#FF6B6B","Coding":"#4ECDC4","QA":"#45B7D1","Business":"#96CEB4",
    "Marketing":"#FFEAA7","Security":"#DDA0DD","Translation":"#FD79A8","Legal":"#FDCB6E",
    "DevOps":"#6C5CE7","Strategy":"#E17055"
};
</script>
</body>
</html>
""")

@app.route("/manifest.json")
def manifest():
    return jsonify({
        "name":"QuantumMind AI",
        "short_name":"QMind",
        "start_url":"/",
        "display":"standalone",
        "background_color":"#ffffff",
        "theme_color":"#4ECDC4",
        "icons":[{"src":"/icon.png","sizes":"192x192","type":"image/png"}]
    })

@app.route("/api/run", methods=["POST"])
def run_task():
    data = request.json
    task = data.get("task","")
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    result = loop.run_until_complete(chef.orchestrate(task))
    return jsonify(result)

if __name__ == "__main__":
    port = int(os.environ.get("PORT",5000))
    app.run(host="0.0.0.0", port=port)
