# GitGrade-hackthon

<!DOCTYPE html>
<html>
<head>
  <title>AI GitHub Repository Analyzer</title>
  <style>
    body { font-family: Arial; background:#f4f6f8; padding:40px; }
    input { width:360px; padding:10px; }
    button { padding:10px 20px; cursor:pointer; }
    #result { margin-top:30px; background:#fff; padding:20px; border-radius:8px; }
    pre { white-space:pre-wrap; }
  </style>
</head>
<body>

<h1>AI GitHub Repository Analyzer</h1>
<p>Paste a public GitHub repository URL</p>

<input id="repoUrl" placeholder="https://github.com/user/repo">
<button onclick="analyzeRepo()">Analyze</button>

<div id="result"></div>

<script>
const GEMINI_API_KEY = "PASTE_YOUR_GEMINI_API_KEY_HERE";

function getRepoDetails(url){
  const p = url.replace("https://github.com/","").split("/");
  return { owner:p[0], repo:p[1] };
}

async function fetchRepoData(owner, repo){
  const r = await fetch(`https://api.github.com/repos/${owner}/${repo}`);
  const repoData = await r.json();
  const c = await fetch(`https://api.github.com/repos/${owner}/${repo}/commits`);
  const commits = await c.json();
  const l = await fetch(`https://api.github.com/repos/${owner}/${repo}/languages`);
  const langs = await l.json();
  return {
    description: repoData.description || "No description",
    commits: commits.length,
    languages: Object.keys(langs)
  };
}

function calculateScore(data){
  let s=0;
  if(data.commits>20) s+=30; else if(data.commits>5) s+=20; else s+=10;
  if(data.languages.length>1) s+=20;
  if(data.description!=="No description") s+=15;
  return s;
}

async function generateAI(data){
  const prompt = `You are a senior software mentor. Analyze this GitHub repository.
Commits: ${data.commits}
Languages: ${data.languages.join(", ")}
Description: ${data.description}
Generate a short evaluation summary and a 4-step improvement roadmap.`;
  const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${GEMINI_API_KEY}`,{
    method:"POST",
    headers:{ "Content-Type":"application/json" },
    body:JSON.stringify({ contents:[{ parts:[{ text:prompt }] }] })
  });
  const out = await res.json();
  return out.candidates[0].content.parts[0].text;
}

async function analyzeRepo(){
  const url = document.getElementById("repoUrl").value;
  document.getElementById("result").innerHTML = "Analyzing...";
  try{
    const {owner,repo} = getRepoDetails(url);
    const data = await fetchRepoData(owner,repo);
    const score = calculateScore(data);
    const ai = await generateAI(data);
    document.getElementById("result").innerHTML = `
      <h2>Score: ${score}/100</h2>
      <p><b>Commits:</b> ${data.commits}</p>
      <p><b>Languages:</b> ${data.languages.join(", ")}</p>
      <h3>AI Evaluation</h3>
      <pre>${ai}</pre>`;
  }catch(e){
    document.getElementById("result").innerHTML = "Error analyzing repository.";
  }
}
</script>

</body>
</html>
