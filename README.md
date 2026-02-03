<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>LOISELLE - Inspection</title>
    <style>
        :root { --primary: #12263a; --accent: #28a745; --red: #d9534f; }
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: sans-serif; background: #0d1b2a; padding: 10px; color: #333; }
        .container { max-width: 600px; margin: auto; background: #fff; border-radius: 20px; padding: 15px; }
        .logo { text-align: center; border-bottom: 2px solid #eee; margin-bottom: 15px; padding-bottom: 10px; }
        .logo h1 { font-size: 2.2em; color: var(--primary); font-weight: 800; }
        .info-card { background: #f8f9fa; padding: 15px; border-radius: 12px; margin-bottom: 15px; }
        label { display: block; font-weight: bold; margin: 5px 0; color: var(--primary); font-size: 0.9em; }
        input, select, textarea { width: 100%; padding: 12px; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 10px; font-size: 16px; }
        
        /* Items d'inspection */
        .item-group { border: 1px solid #eee; border-radius: 12px; margin-bottom: 10px; padding: 12px; background: #fff; }
        .item-main { display: flex; align-items: center; justify-content: space-between; }
        .status-options { display: flex; gap: 8px; }
        .status-options input { display: none; }
        .status-options label { padding: 10px 15px; border-radius: 8px; border: 1px solid #ccc; cursor: pointer; font-size: 0.85em; font-weight: bold; background: #fff; }
        
        .status-options input:checked + label.ok { background: var(--accent); color: white; border-color: var(--accent); }
        .status-options input:checked + label.defect { background: var(--red); color: white; border-color: var(--red); }

        /* Zone de preuve (Photo/Note) - cachée par défaut */
        .defect-proof { display: none; margin-top: 15px; padding-top: 15px; border-top: 1px dashed #ccc; background: #fff5f5; border-radius: 0 0 10px 10px; padding: 10px; }
        
        .sig-container { border: 2px dashed #ccc; border-radius: 10px; background: #fff; height: 150px; }
        #sig-canvas { width: 100%; height: 100%; touch-action: none; }
        .btn-submit { background: #ffc107; width: 100%; padding: 20px; border: none; border-radius: 10px; font-size: 18px; font-weight: bold; color: #12263a; margin-top: 20px; cursor: pointer; }
        
        #receipt { display: none; text-align: center; }
        .receipt-box { border: 2px solid var(--accent); border-radius: 15px; padding: 20px; background: #f0fff4; text-align: left; margin-top: 20px; }
    </style>
</head>
<body>

<div class="container" id="main-form">
    <div class="logo"><h1>LOISELLE</h1><p>INSPECTION VÉHICULE</p></div>
    <form id="inspectionForm" action="https://formspree.io/f/xdadypze" method="POST" enctype="multipart/form-data">
        
        <div class="info-card">
            <label>👤 Employé (Employee) :</label>
            <input type="text" name="Employe" id="emp_name" placeholder="Nom" required>
            <label>🔢 Plaque d'immatriculation (Plate) :</label>
            <input type="text" name="Plaque" id="plate_id" placeholder="L123456" required>
            <label>🚛 Unité (Unit) :</label>
            <input type="text" name="Equipement" id="equip_id" required>
            <label>📅 Date :</label>
            <input type="date" name="Date_Inspection" id="inspect_date" required>
        </div>

        <div id="itemsList"></div>

        <div class="info-card">
            <label>✍️ Signature :</label>
            <div class="sig-container"><canvas id="sig-canvas"></canvas></div>
            <input type="hidden" name="Signature_Data" id="sig-data">
        </div>

        <button type="submit" class="btn-submit">🚀 ENVOYER / SUBMIT</button>
    </form>
</div>

<div class="container" id="receipt">
    <div class="logo"><h1>LOISELLE</h1></div>
    <div class="receipt-box">
        <h2 style="color:var(--accent); text-align:center;">✅ INSPECTION COMPLÉTÉE</h2>
        <hr style="margin:15px 0;">
        <p><strong>Employé :</strong> <span id="r-name"></span></p>
        <p><strong>Plaque :</strong> <span id="r-plate"></span></p>
        <p><strong>Date :</strong> <span id="r-date"></span></p>
        <p style="margin-top:10px; font-weight:bold; color:var(--accent);">STATUT : CONFORME / COMPLIANT</p>
    </div>
    <button onclick="location.reload()" class="btn-submit" style="background:#eee; margin-top:30px;">NOUVELLE INSPECTION</button>
</div>

<script>
    const items = ["Niveaux", "Freins", "Pneus", "Lumières", "Direction", "Vitre/Miroirs", "Sécurité", "Fuites"];
    const list = document.getElementById('itemsList');

    items.forEach((item, i) => {
        list.innerHTML += `
            <div class="item-group">
                <div class="item-main">
                    <strong>${item}</strong>
                    <div class="status-options">
                        <input type="radio" name="${item}" value="OK" id="ok${i}" required onclick="toggleDefect(${i}, false)">
                        <label for="ok${i}" class="ok">OK</label>
                        <input type="radio" name="${item}" value="DEFAUT" id="def${i}" required onclick="toggleDefect(${i}, true)">
                        <label for="def${i}" class="defect">DEF</label>
                    </div>
                </div>
                <div class="defect-proof" id="proof${i}">
                    <label>📸 Photo du problème (${item}) :</label>
                    <input type="file" name="Photo_${item}" accept="image/*" capture="environment">
                    <label>📝 Note sur la défectuosité :</label>
                    <textarea name="Note_${item}" rows="2" placeholder="Expliquez le problème..."></textarea>
                </div>
            </div>`;
    });

    function toggleDefect(index, isDefect) {
        document.getElementById(`proof${index}`).style.display = isDefect ? 'block' : 'none';
    }

    document.getElementById('inspect_date').valueAsDate = new Date();

    const canvas = document.getElementById('sig-canvas');
    const ctx = canvas.getContext('2d');
    let drawing = false;

    function resize() { canvas.width = canvas.offsetWidth; canvas.height = canvas.offsetHeight; ctx.lineWidth=2; }
    window.onload = resize;

    function getPos(e) {
        const rect = canvas.getBoundingClientRect();
        const client = e.touches ? e.touches[0] : e;
        return { x: client.clientX - rect.left, y: client.clientY - rect.top };
    }

    canvas.addEventListener('touchstart', (e) => { drawing=true; ctx.beginPath(); const p=getPos(e); ctx.moveTo(p.x, p.y); e.preventDefault(); }, {passive:false});
    canvas.addEventListener('touchmove', (e) => { if(!drawing) return; const p=getPos(e); ctx.lineTo(p.x, p.y); ctx.stroke(); e.preventDefault(); }, {passive:false});
    window.addEventListener('touchend', () => drawing=false);

    document.getElementById("inspectionForm").onsubmit = function() {
        document.getElementById('sig-data').value = canvas.toDataURL("image/jpeg", 0.5);
        document.getElementById('r-name').textContent = document.getElementById('emp_name').value;
        document.getElementById('r-plate').textContent = document.getElementById('plate_id').value;
        document.getElementById('r-date').textContent = new Date().toLocaleString();
        document.getElementById('main-form').style.display = 'none';
        document.getElementById('receipt').style.display = 'block';
        window.scrollTo(0,0);
    };
</script>
</body>
</html>
