<script>  
  import * as d3 from "d3";
  import paises from "/src/data/countries.csv";

  // Cantidad
  const minmaxKoreans = d3.extent(paises, (d) => d.CantidadSurcoreanos);
  var diamCircles = d3.scaleRadial().domain(minmaxKoreans).range([5, 160]);

  // Distancia
  const maxDistance = d3.max(paises, (d) => d.Distancia);
  var xScale = d3.scaleLinear().domain([0, maxDistance]).range([0, 900]);

  // Continente  
  const colorContinent = d3.scaleOrdinal()
    .domain(["Asia", "Oceanía", "Europa", "América del Norte", "América del Sur"])
    .range(["hsl(0, 0%, 90%)", "hsl(0, 0%, 70%)", "hsl(0, 0%, 50%)", "hsl(0, 0%, 30%)", "hsl(0, 0%, 5%)"]);

  // Razon
  const colorReason = d3.scaleOrdinal()
    .domain(["Económica", "Histórica", "Educativa", "Otro"])
    .range(["#32CD32", "#EE1010", "#1E90FF", "#B600D6"]);

  // Filtros
  let activeFilters = {
    continente: "todos",
    razon: "todos"
  };

  // Creacion de los circulos
  let nodes = paises.map(function(d) {
    return {
      radius: diamCircles(d.CantidadSurcoreanos),
      position: xScale(d.Distancia),
      color: colorContinent(d.Continente),
      strokeColor: colorReason(d.Razon),
      strokeWidth: 3,
      Ranking: d.Ranking,
      Pais: d.Pais,
      CantidadSurcoreanos: d.CantidadSurcoreanos,
      Continente: d.Continente,
      Distancia: d.Distancia,
      Razon: d.Razon,
      opacity: 1
    };
  });

  // Filtrar por Continente
  function filterByContinente(continente) {
    activeFilters.continente = continente;
    applyFilters();
    updateButtonStates("continente", continente);
  }

  // Filtrar por Razon
  function filterByRazon(razon) {
    activeFilters.razon = razon;
    applyFilters();
    updateButtonStates("razon", razon);
  }

  // Aplicar filtros
  function applyFilters() {
    nodes.forEach(node => {
      const continenteMatch = activeFilters.continente === "todos" || node.Continente === activeFilters.continente;
      const razonMatch = activeFilters.razon === "todos" || node.Razon === activeFilters.razon;
      node.opacity = continenteMatch && razonMatch ? 1 : 0.2;
    });
    simulation.alpha(1).restart();
  }

  // Resetear filtros
  function resetFilters() {
    activeFilters = {
      continente: "todos",
      razon: "todos"
    };
    nodes.forEach(node => {
      node.opacity = 1;
    });
    updateButtonStates("none", "none");
    simulation.alpha(1).restart();
  }

  // Actualizar botones
  function updateButtonStates(filterType, value) {
    document.querySelectorAll(`.filter_button[data-filter="${filterType}"]`).forEach(button => {
        button.classList.remove('active');
    });
    if (filterType !== "none") {
        document.querySelector(`[data-filter="${filterType}"][data-value="${value}"]`)?.classList.add('active');
    }
  }

  // Simulacion de fuerzas
  var simulation = d3.forceSimulation(nodes)
    .force('charge', d3.forceManyBody().strength(0.1))
    .force('x', d3.forceX().x(function(d) {
      return d.position + 100;
    }))
    .force('y', d3.forceY().y(function(d) {
      return 100;
    }))
    .force('collision', d3.forceCollide().radius(function(d) {
      return d.radius + 2;
    }))
    .on('tick', ticked);

  function ticked() {
    const tooltip = d3.select('.tooltip');
    const formatNumber = (num) => {
      return d3.format(",.0f")(num).replace(/,/g, '.');
    };

    d3.select('svg g')
      .selectAll('circle')
      .data(nodes)
      .join('circle')
      .attr('r', function(d) {
        return d.radius;
      })
      .style('fill', function(d) {
        return d.color;
      })
      .style('stroke-width', function(d) {
        return d.strokeWidth;
      })
      .style('stroke', function(d) {
        return d.strokeColor;
      })
      .style('opacity', function(d) {
        return d.opacity;
      })
      .attr('cx', function(d) {
        return d.x;
      })
      .attr('cy', function(d) {
        return d.y;
      })
      .on('mouseover', function(event, d) {
        if (d.opacity === 1) {
          d3.select(this)
            .transition()
            .duration(200)
            .style('opacity', 0.7);
          
          tooltip.transition()
            .duration(200)
            .style('opacity', 0.9);
          
          tooltip.html(`
            <strong>Ranking:</strong> ${formatNumber(d.Ranking)}<br>
            <strong>País:</strong> ${d.Pais}<br>
            <strong>Cantidad de Surcoreanos:</strong> ${formatNumber(d.CantidadSurcoreanos)}<br>
            <strong>Continente:</strong> ${d.Continente}<br>
            <strong>Distancia:</strong> ${formatNumber(d.Distancia)} km<br>
            <strong>Razón:</strong> ${d.Razon}`)
            .style('left', (event.pageX + 5) + 'px')
            .style('top', (event.pageY - 28) + 'px');
        }
      })
      .on('mousemove', function(event) {
        tooltip.style('left', (event.pageX + 5) + 'px')
          .style('top', (event.pageY - 28) + 'px');
      })
      .on('mouseout', function(event, d) {
        if (d.opacity === 1) {
          d3.select(this)
            .transition()
            .duration(200)
            .style('opacity', d.opacity);
          
          tooltip.transition()
            .duration(500)
            .style('opacity', 0);
        }
      });
  }
</script>




<main>
  <!-- Emblema Corea del Sur -->
  <div style="text-align: center">
    <img id="southKoreaEmblem" src="./images/Emblem_of_South_Korea.png" alt="Emblema de Corea del Sur">
  </div>
  
  <h1 class="header">La Diáspora Surcoreana</h1>

  <!-- Referencia Cantidad -->
  <div class="ref_Cantidad">
    <h4 class="ref_Cantidad_header">Cantidad</h4>
      <div class="ref_circulo_wrapper">
        <svg width="200" height="200"><circle cx="100" cy="100" r="10" fill="hsl(0, 0%, 100%)" /></svg>
        <p>10k</p>
      </div>
      <div class="ref_circulo_wrapper" id="circulo_150k">
        <svg width="200" height="200"><circle cx="100" cy="100" r="45" fill="hsl(0, 0%, 100%)" /></svg>
        <p>150k</p>
      </div>
      <div class="ref_circulo_wrapper">
        <svg width="200" height="200"><circle cx="100" cy="100" r="100" fill="hsl(0, 0%, 100%)" /></svg>
        <p>800k</p>
      </div>
  </div>
  
  <!-- Referencias Continente y Razon con botones y filtros -->
  <div class="button_container">
    <!-- Botones Continente -->
    <div class="filter_group">
      <h4>Continente</h4>
      <div class="filter_group_buttons">
        <button class="filter_button" data-filter="continente" data-value="Asia" on:click={() => { filterByContinente("Asia"); updateButtonStates("continente", "Asia"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="hsl(0, 0%, 90%)" /></svg>
          <p>Asia</p>
        </button>
        <button class="filter_button" data-filter="continente" data-value="Oceanía" on:click={() => { filterByContinente("Oceanía"); updateButtonStates("continente", "Oceanía"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="hsl(0, 0%, 70%)" /></svg>
          <p>Oceanía</p>
        </button>
        <button class="filter_button" data-filter="continente" data-value="Europa" on:click={() => { filterByContinente("Europa"); updateButtonStates("continente", "Europa"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="hsl(0, 0%, 50%)" /></svg>
          <p>Europa</p>
        </button>
        <button class="filter_button" data-filter="continente" data-value="América del Norte" on:click={() => { filterByContinente("América del Norte"); updateButtonStates("continente", "América del Norte"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="hsl(0, 0%, 30%)" /></svg>
          <p>América del Norte</p>
        </button>
        <button class="filter_button" data-filter="continente" data-value="América del Sur" on:click={() => { filterByContinente("América del Sur"); updateButtonStates("continente", "América del Sur"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="hsl(0, 0%, 5%)" /></svg>
          <p>América del Sur</p>
        </button>
      </div>
    </div>
    <!-- Botones Razon -->
    <div class="filter_group">
      <h4>Razón</h4>
      <div class="filter_group_buttons">
        <button class="filter_button" data-filter="razon" data-value="Económica" on:click={() => { filterByRazon("Económica"); updateButtonStates("razon", "Económica"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="white" stroke-width="3" stroke="#32CD32" /></svg>
          <p>Económica</p>
        </button>
        <button class="filter_button" data-filter="razon" data-value="Histórica" on:click={() => { filterByRazon("Histórica"); updateButtonStates("razon", "Histórica"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="white" stroke-width="3" stroke="#EE1010" /></svg>
          <p>Histórica</p>
        </button>
        <button class="filter_button" data-filter="razon" data-value="Educativa" on:click={() => { filterByRazon("Educativa"); updateButtonStates("razon", "Educativa"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="white" stroke-width="3" stroke="#1E90FF" /></svg>
          <p>Educativa</p>
        </button>
        <button class="filter_button" data-filter="razon" data-value="Otro" on:click={() => { filterByRazon("Otro"); updateButtonStates("razon", "Otro"); }}>
          <svg width="100" height="100"><circle cx="50" cy="50" r="30" fill="white" stroke-width="3" stroke="#B600D6" /></svg>
          <p>Otro</p>
        </button>
      </div>
    </div>
    <!-- Boton Resetear -->
    <div class="filter_group" style="position: relative; top:30px">
      <button id="button_ResetAll" on:click={resetFilters}>Mostrar Todo</button>
    </div>
  </div>

  <!-- Gráfico de circulos con simulacion-->
  <div class="grafico_container">
    <svg id="circulos" width="1300" height="500">
      <g transform="translate(150, 150)"></g>
    </svg>
  </div>
  <div class="tooltip" style="opacity: 0; position: absolute; background: white; border: 1px solid black; padding: 5px; pointer-events: none;"></div>
  <!-- Escala Distancia -->
  <div class="escala">
    <div class="marca_num" style="left:0%">0 km</div>
    <div class="marca_num" style="left:25%">5.000</div>
    <div class="marca_num" style="left:50%">10.000</div>
    <div class="marca_num" style="left:75%">15.000</div>
    <div class="marca_num" style="left:100%">20.000</div>
    <h6 style="margin-top: 40px">Distancia a Corea del Sur</h6>
  </div>

  <footer>
    <p>Ian David Yong Joon Kim</p>
    <p>
      <a href="https://github.com/iannkimutdt" target="_blank">GitHub</a> |
      <a href="https://www.linkedin.com/in/iandavidyongjoonkim/" target="_blank">LinkedIn</a> 
    </p>
    <p>Visualización de Datos - Universidad Torcuato Di Tella</p>
  </footer>
</main>




<style>
  #southKoreaEmblem {
    width: 100px;
    height: 100px;
  }


  .header {
    font-family: "Garamond";
    font-weight: bold;
    text-align: center;
    margin-bottom: 100px;
  }
  
  /* Refencia Cantidad */
  .ref_Cantidad {
    display: flex;
    justify-content: center;
    margin-bottom: 40px;
    text-align: center;
  }
  .ref_Cantidad_header {
    position: relative;
    top: 70px;
    font-weight: bold;
  }
  #circulo_150k {
    position: relative;
    left:-30px;
  }

  /* Referencias Continente y Razon con botones y filtros  */
  .filter_group h4 {
    position: relative;
    top: 10px;
    text-align: center;
    font-weight: bold;
  }
  .filter_group_buttons {
    display: flex;
    text-align: center;
    align-items: center;
    margin-right: 30px;
  }
  .filter_group p {
    position: relative;
    top: -25px;
  }
  .button_container {
    position: relative;
    left: -70px;
    width: 1350px;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-around; 
  }
  button {
    border-radius: 4px;
    background-color: transparent;
    font-size: 14px;
    height: 140px;
    color: white;
    font-family: "Garamond";
  }
  button:hover {
    background-color: #f0f0f0;
    border-color: #f0f0f0;
    color: #0056b3;
  }
  button:focus {
    background-color: #f0f0f0;
    color: #0F64CD;
    border-color: #f0f0f0;
    outline: none;
  }

  /* Gráfico de circulos con simulacion */
  .grafico_container {
    position: relative;
    top: -30px;
    left: -50px;
  }
  .tooltip {
    opacity: 0;
    position: absolute;
    background: white;
    border: 1px solid black;
    padding: 5px;
    pointer-events: none;
    border-radius: 5px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
    z-index: 1;
    color: black;
  }
  .escala {
    width: 1200px;
    height: 2px; 
    background-color: black; 
    position: relative;
    top: -40px;
    display: flex;
  } 
  .marca_num {
    position: absolute;
    margin-top: 5px;
  }


  footer {
    text-align: center;
    font-size: 15px;
    margin-top: 120px;
  }
  footer a {
    color: #CD2E3A;
    margin: 0 3px;
  }
  footer a:hover {
    color: white;
  }
</style>

