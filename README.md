
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Sorteio Adventista</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.13.216/pdf.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f7f7f7;
      margin: 0; padding: 20px;
      text-align: center;
    }
    header {
      display: flex;
      justify-content: center;
      gap: 40px;
      margin-bottom: 20px;
    }
    header img {
      height: 80px;
      border-radius: 8px;
    }
    h1 {
      color: #003366;
      margin-bottom: 10px;
    }
    .card {
      background: #fff;
      padding: 20px;
      margin: 20px auto;
      max-width: 600px;
      border-radius: 12px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.2);
      text-align: left;
    }
    label { display: block; margin: 10px 0 5px; font-weight: bold; }
    input, button, textarea {
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 6px;
      width: 100%;
      margin-bottom: 10px;
    }
    button {
      background: #003366;
      color: #fff;
      cursor: pointer;
      transition: 0.3s;
    }
    button:hover { background: #0055aa; }
    #resultado {
      margin-top: 20px;
      font-size: 18px;
      color: #006600;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <header>
    <div>
      <label>Logo Educação Adventista</label>
      <input type="file" id="logoEduInput" accept="image/*" />
      <img id="logoEdu" alt="Logo Educação Adventista" />
    </div>
    <div>
      <label>Logo Igreja Adventista</label>
      <input type="file" id="logoIgrejaInput" accept="image/*" />
      <img id="logoIgreja" alt="Logo Igreja Adventista" />
    </div>
  </header>

  <h1>Sorteio Adventista</h1>

  <div class="card">
    <label for="nomes">Digite nomes (um por linha):</label>
    <textarea id="nomes" rows="6" placeholder="Exemplo: Maria&#10;João&#10;Ana"></textarea>

    <label>Ou envie um arquivo (Excel/CSV/PDF):</label>
    <input type="file" id="fileInput" accept=".xlsx,.xls,.csv,.pdf" />

    <label for="qtd">Quantidade de vencedores:</label>
    <input type="number" id="qtd" value="1" min="1" />

    <button onclick="sortear()">Sortear</button>
    <div id="resultado"></div>
  </div>

  <script>
    // Upload logos
    document.getElementById("logoEduInput").addEventListener("change", e => {
      const f = e.target.files[0];
      if (f) document.getElementById("logoEdu").src = URL.createObjectURL(f);
    });
    document.getElementById("logoIgrejaInput").addEventListener("change", e => {
      const f = e.target.files[0];
      if (f) document.getElementById("logoIgreja").src = URL.createObjectURL(f);
    });

    let lista = [];

    document.getElementById("fileInput").addEventListener("change", async e => {
      const file = e.target.files[0];
      if (!file) return;
      const ext = file.name.split(".").pop().toLowerCase();
      if (["xlsx","xls","csv"].includes(ext)) {
        const data = await file.arrayBuffer();
        const wb = XLSX.read(data, {type:"array"});
        const sheet = wb.Sheets[wb.SheetNames[0]];
        const rows = XLSX.utils.sheet_to_json(sheet,{header:1});
        rows.flat().forEach(v => { if(v) lista.push(v); });
      } else if (ext === "pdf") {
        const data = new Uint8Array(await file.arrayBuffer());
        const pdf = await pdfjsLib.getDocument({data}).promise;
        for (let i=1;i<=pdf.numPages;i++){
          const page = await pdf.getPage(i);
          const text = await page.getTextContent();
          text.items.forEach(it=> lista.push(it.str));
        }
      }
      alert("Arquivo carregado com sucesso!");
    });

    function sortear() {
      let nomesText = document.getElementById("nomes").value.split(/[\n,;]/).map(n=>n.trim()).filter(Boolean);
      let todos = [...nomesText, ...lista].map(n=>n.trim()).filter(Boolean);
      todos = [...new Set(todos)];
      const qtd = parseInt(document.getElementById("qtd").value);
      if (todos.length === 0) { alert("Nenhum nome válido."); return; }
      if (qtd>todos.length) { alert("Quantidade maior que a lista."); return; }
      let vencedores = [];
      let pool = [...todos];
      for(let i=0;i<qtd;i++){
        const idx = Math.floor(Math.random()*pool.length);
        vencedores.push(pool.splice(idx,1)[0]);
      }
      document.getElementById("resultado").innerHTML =
        "🎉 Vencedores:<br>" + vencedores.join("<br>");
    }
  </script>
</body>
</html>
