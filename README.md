# Asya-3
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8" />
  <title>Asya Ülkeleri Etkileşimli Harita</title>
  <style>
    body { margin: 0; overflow: hidden; font-family: sans-serif; }
    #infoBox {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(255,255,255,0.9);
      padding: 15px;
      border-radius: 8px;
      max-width: 300px;
      display: none;
    }
    h2 { margin-top: 0; }
  </style>
</head>
<body>
  <div id="infoBox">
    <h2 id="countryName"></h2>
    <p><strong>Kuruluş:</strong> <span id="founding"></span></p>
    <p><strong>Başkent:</strong> <span id="capital"></span></p>
    <p><strong>Nüfus:</strong> <span id="population"></span></p>
    <p><strong>İklim:</strong> <span id="climate"></span></p>
    <p><strong>Yer Altı Kaynakları:</strong> <span id="resources"></span></p>
    <p><strong>Tarihi ve Doğal Yerler:</strong> <span id="landmarks"></span></p>
  </div>

  <!-- Globe.gl ve gerekli kütüphaneler -->
  <script src="https://unpkg.com/three@0.152.2/build/three.min.js"></script>
  <script src="https://unpkg.com/globe.gl"></script>
  <script src="https://unpkg.com/topojson@3"></script>

  <script>
    const countriesData = {
      "China": {
        founding: "1 Ekim 1949",
        capital: "Pekin",
        population: "1.4 milyar",
        climate: "Karasal, Muson",
        resources: "Kömür, demir, nadir elementler",
        landmarks: "Çin Seddi, Yasak Şehir"
      },
      "India": {
        founding: "15 Ağustos 1947",
        capital: "Yeni Delhi",
        population: "1.4 milyar",
        climate: "Tropikal, muson",
        resources: "Kömür, demir cevheri, boksit",
        landmarks: "Tac Mahal, Ganj Nehri"
      },
      "Japan": {
        founding: "11 Şubat M.Ö. 660",
        capital: "Tokyo",
        population: "125 milyon",
        climate: "Ilıman",
        resources: "Balık, nadir mineraller (ithalat ağırlıklı)",
        landmarks: "Fuji Dağı, Kyoto Tapınakları"
      }
    };

    const world = Globe()
      (document.body)
      .globeImageUrl('//unpkg.com/three-globe/example/img/earth-dark.jpg')
      .bumpImageUrl('//unpkg.com/three-globe/example/img/earth-topology.png')
      .backgroundColor('#000')
      .pointOfView({ lat: 30, lng: 90, altitude: 2.5 }, 4000);

    fetch('https://unpkg.com/world-atlas@1/world/110m.json')
      .then(res => res.json())
      .then(worldData => {
        const countries = topojson.feature(worldData, worldData.objects.countries).features;

        // Sadece Çin, Hindistan ve Japonya'yı filtrele
        const targetCountries = countries.filter(d =>
          ["China", "India", "Japan"].includes(d.properties.name)
        );

        world
          .polygonsData(targetCountries)
          .polygonAltitude(0.06)
          .polygonCapColor(d => {
            switch (d.properties.name) {
              case "China": return "rgba(255, 0, 0, 0.8)";
              case "India": return "rgba(0, 255, 0, 0.8)";
              case "Japan": return "rgba(0, 0, 255, 0.8)";
              default: return "rgba(200, 200, 200, 0.8)";
            }
          })
          .polygonSideColor(() => 'rgba(0, 100, 0, 0.15)')
          .polygonStrokeColor(() => '#111')
          .onPolygonClick((polygon) => {
            const countryName = polygon.properties.name;
            if (countriesData[countryName]) {
              document.getElementById('infoBox').style.display = 'block';
              document.getElementById('countryName').textContent = countryName;
              document.getElementById('founding').textContent = countriesData[countryName].founding;
              document.getElementById('capital').textContent = countriesData[countryName].capital;
              document.getElementById('population').textContent = countriesData[countryName].population;
              document.getElementById('climate').textContent = countriesData[countryName].climate;
              document.getElementById('resources').textContent = countriesData[countryName].resources;
              document.getElementById('landmarks').textContent = countriesData[countryName].landmarks;
            }
          });

        // Ülke isimlerini etiket olarak ekle
        const labels = targetCountries.map(d => ({
          lat: d.properties.latitude || 0,
          lng: d.properties.longitude || 0,
          text: d.properties.name
        }));

        world.labelsData(labels)
          .labelLat(d => d.lat)
          .labelLng(d => d.lng)
          .labelText(d => d.text)
          .labelSize(1.5)
          .labelColor(() => 'white')
          .labelResolution(2);
      });
  </script>
</body>
</html>
