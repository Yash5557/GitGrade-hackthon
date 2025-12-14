# GitGrade-hackthon
here is the video link - https://youtu.be/rL0EhIwqIg0
<html>
<head>
  <title>GitHub Repository Analyzer</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: "Segoe UI", system-ui, sans-serif;
      background: linear-gradient(135deg, #eef2f7, #dbeafe);
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #1f2937;
    }
    .container {
      background: #ffffff;
      width: 100%;
      max-width: 820px;
      padding: 32px;
      border-radius: 16px;
      box-shadow: 0 20px 40px rgba(0,0,0,0.1);
    }
    h1 {
      text-align: center;
      margin-bottom: 8px;
      font-size: 28px;
      font-weight: 700;
    }
    .subtitle {
      text-align: center;
      color: #6b7280;
      margin-bottom: 24px;
    }
    .input-box {
      display: flex;
      gap: 12px;
      margin-bottom: 28px;
    }
    input {
      flex: 1;
      padding: 14px 16px;
      border-radius: 10px;
      border: 1px solid #d1d5db;
      font-size: 16px;
      outline: none;
    }
    input:focus {
      border-color: #2563eb;
    }
    button {
      padding: 14px 24px;
      border-radius: 10px;
      border: none;
      background: #2563eb;
      color: white;
      font-size: 16px;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.2s ease;
    }
    button:hover {
      background: #1d4ed8;
    }
    .card {
      background: #f9fafb;
      padding: 24px;
      border-radius: 14px;
      margin-top: 20px;
      border: 1px solid #e5e7eb;
    }
    .score {
      font-size: 36px;
      font-weight: 800;
      color: #16a34a;
      margin-bottom: 10px;
    }
    .badges span {
      display: inline-block;
      background: #e0e7ff;
      color: #1e3a8a;
      padding: 6px 12px;
      border-radius: 999px;
      font-size: 14px;
      margin-right: 8px;
      margin-bottom: 8px;
    }
    h3 {
      margin-top: 20px;
      margin-bottom: 8px;
      font-size: 18px;
    }
    pre {
      background: white;
      padding: 16px;
      border-radius: 10px;
      border: 1px solid #e5e7eb;
      white-space: pre-wrap;
      line-height: 1.6;
    }
    .loading {
      text-align: center;
      font-weight: 600;
      color: #2563eb;
    }
    @media(max-width: 600px){
      .input-box { flex-direction: column; }
      button { width: 100%; }
    }
  </style>
</head>

<body>
  <div class="container">
    <h1>GitHub Repository Analyzer</h1>
    <p class="subtitle">Instant AI-powered evaluation of any GitHub project</p>

    <div class="input-box">
      <input id="repoUrl" placeholder="https://github.com/user/repository">
      <button onclick="analyzeRepo()">Analyze</button>
    </div>

    <div id="result"></div>
  </div>

<script>
const GEMINI_API_KEY = "AIzaSyDuQdqEJQnh4a6tuNXPbpNyLSdEl16aUbc";

function getRepoDetails(url){
  if(!url.includes("github.com")) throw "Invalid URL";
  const p = url.replace("https://github.com/","").split("/");
  return { owner:p[0], repo:p[1] };
}

async function fetchRepoData(owner, repo){
  const res = await fetch(`https://api.github.com/repos/${owner}/${repo}`, {
    headers: { "Accept": "application/vnd.github.v3+json" }
  });
  const data = await res.json();

  if(data.message) throw data.message;

  return {
    name: data.name,
    description: data.description || "No description",
    stars: data.stargazers_count,
    forks: data.forks_count,
    language: data.language || "Unknown",
    size: data.size,
    updated: data.updated_at
  };
}

function calculateScore(d){
  let score = 0;
  if(d.description !== "No description") score += 20;
  if(d.stars > 0) score += 20;
  if(d.forks > 0) score += 10;
  if(d.size > 100) score += 20;
  if(d.language !== "Unknown") score += 20;
  return score;
}

async function generateAI(data){
  if(!GEMINI_API_KEY || GEMINI_API_KEY.includes("AIzaSyDuQdqEJQnh4a6tuNXPbpNyLSdEl16aUbc"))
    return "AI disabled.";

  const prompt = `
Evaluate this GitHub project as a senior software mentor.

Project: ${data.name}
Language: ${data.language}
Stars: ${data.stars}
Description: ${data.description}

Give:
1. Short summary
2. 4-step improvement roadmap
`;

  const res = await fetch(
    `https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${GEMINI_API_KEY}`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        contents: [{ parts: [{ text: prompt }] }]
      })
    }
  );

  const out = await res.json();
  return out?.candidates?.[0]?.content?.parts?.[0]?.text
    || "AI feedback unavailable.";
}

async function analyzeRepo(){
  const url = document.getElementById("repoUrl").value.trim();
  const result = document.getElementById("result");
  result.innerHTML = "<p class='loading'>Analyzing…</p>";

  try{
    const {owner,repo} = getRepoDetails(url);
    const data = await fetchRepoData(owner,repo);
    const score = calculateScore(data);
    const ai = await generateAI(data);

    result.innerHTML = `
      <div class="card">
        <div class="score">${score}/100</div>
        <p><b>Project:</b> ${data.name}</p>
        <p><b>Language:</b> ${data.language}</p>
        <p><b>Stars:</b> ⭐ ${data.stars}</p>
        <p><b>Description:</b> ${data.description}</p>
        <h3>AI Evaluation</h3>
        <pre>${ai}</pre>
      </div>
    `;
  }catch(err){
    result.innerHTML = `<p style="color:red;">${err}</p>`;
  }
}
</script>

</body>
</html>
