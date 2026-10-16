# Guía de Despliegue Web IaaS en Oracle Cloud Infrastructure (OCI) con Nginx y DuckDNS

**Estudiante:** Alejandro Aguilera  
**Asignatura:** Infraestructura y Cloud  
**Proveedor Cloud:** Oracle Cloud Infrastructure (OCI) - Región Brazil East (São Paulo) `sa-saopaulo-1`  
**Instancia (VM):** `pagina-web-FA` (Oracle Linux Server)  
**Dirección IPv4 Pública:** `147.15.45.117`  
**Subdominio DuckDNS:** `http://apexcorse-oci.duckdns.org` *(Reemplazar por tu subdominio exacto)*

---

## 1. Reutilización de la Infraestructura de Red (VCN) y Reglas de Seguridad

Para el despliegue se reutilizó la infraestructura de red virtual creada en el taller inicial dentro del compartimento `Infraestructura_y_Cloud.`:

1. **Virtual Cloud Network (VCN):** `red-infraestructura` con bloque CIDR IPv4 `10.0.0.0/16`.
2. **Subred Pública Regional:** `subred-web` con bloque CIDR IPv4 `10.0.0.0/24`.
3. **Configuración de Lista de Seguridad (`Default Security List for red-infraestructura`):**
   * Se accedió a **Networking** > **Virtual cloud networks** > **red-infraestructura** > **Security Lists**.
   * Se verificó/agregó la regla de entrada (**Ingress Rule**) para permitir tráfico HTTP desde cualquier lugar de Internet:
     * **Source CIDR:** `0.0.0.0/0`
     * **IP Protocol:** `TCP`
     * **Destination Port Range:** `80`
     * **Description:** `Permitir tráfico HTTP en el puerto 80`

### Evidencias del Paso 1:
![Configuración de la VCN](img/vcn_config.png)
![Lista de Seguridad por Defecto](img/security_list.png)

---

## 2. Creación de la Nueva Máquina Virtual en OCI Compute

1. En el menú principal de OCI se seleccionó **Compute** > **Instances** > **Create instance**.
2. Se definieron los siguientes parámetros de aprovisionamiento:
   * **Name:** `pagina-web-FA`
   * **Create in compartment:** `Infraestructura_y_Cloud.`
   * **Placement (Availability domain):** `AD 1 (HPFR:SA-SAOPAULO-1-AD-1)`
   * **Image and shape:** Oracle Linux (imagen oficial de OCI).
3. En la sección **Primary VNIC / Networking**:
   * **Primary network:** `Select existing virtual cloud network` -> `red-infraestructura`.
   * **Subnet:** `Select existing subnet` -> `subred-web (regional)`.
   * **Private IPv4 address:** `Automatically assign private IPv4 address`.
   * **Public IPv4 address:** `Automatically assign public IPv4 address`.
4. En la sección **Add SSH keys**, se descargó la clave privada `web-key.key` para la conexión segura por consola.
5. Una vez aprovisionada, la instancia pasó a estado **Running** con la dirección IP pública **`147.15.45.117`**.

### Evidencias del Paso 2:
![Formulario de Creación de Instancia](img/create_instance.png)
![Configuración de VNIC y Subred](img/vnic_config.png)
![Instancia en Estado Running](img/instance_running.png)

---

## 3. Conexión Remota mediante SSH desde Oracle Cloud Shell

1. Se abrió la terminal integrada **Oracle Cloud Shell** en la consola de OCI (`alejoaguil@cloudshell:~ (sa-saopaulo-1)$`).
2. Se verificó el directorio `.ssh` y se subió la llave privada `web-key.key` al directorio personal.
3. Se asignaron permisos restrictivos de solo lectura (`chmod 400`) sobre la llave privada y se inició la conexión SSH con el usuario por defecto `opc` hacia la IP `147.15.45.117`:

```bash
# Verificación del directorio SSH
ls -la ~/.ssh

# Restricción de permisos sobre la llave privada SSH
chmod 400 ~/web-key.key

# Conexión SSH a la máquina virtual Oracle Linux
ssh -i ~/web-key.key opc@147.15.45.117
```

### Evidencias del Paso 3:
![Inicio de Oracle Cloud Shell](img/cloudshell_init.png)
![Verificación de Directorio SSH](img/ssh_dir.png)
![Conexión SSH Exitosa a la VM](img/ssh_connection.png)

---

## 4. Instalación de Nginx y Apertura del Firewall Interno

Una vez dentro de la máquina virtual (`[opc@pagina-web-fa ~]$`), se instaló el servidor web Nginx, se habilitó como servicio del sistema y se permitió el tráfico HTTP en el cortafuegos interno (`firewalld`) de Oracle Linux:

```bash
# Instalación del servidor web Nginx
sudo dnf install nginx -y

# Habilitación e inicio automático del servicio Nginx
sudo systemctl enable --now nginx

# Apertura del puerto 80 (HTTP) en el firewall local del sistema operativo
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload

# Comprobación del estado del servicio Nginx
sudo systemctl status nginx
```

### Evidencia del Paso 4:
![Estado Activo de Nginx](img/nginx_status.png)

---

## 5. Creación y Publicación del Portal Web Propio

Se construyó el portal web **ApexCorse | Motociclismo de Alto Rendimiento**, el cual cumple con:
* **Tres secciones definidas:**
  1. *01 - Ingeniería & Dinámica de Competición* (información técnica y fotografía de circuito).
  2. *02 - Simulador de Potencia & Rendimiento* (calculadora interactiva en JavaScript).
  3. *03 - Arquitectura de Despliegue en Cloud* (detalles del servidor IaaS).
* **Imágenes:** Fotografía principal en el encabezado (*Hero Banner*) y fotografía técnica en la primera sección.
* **Estilos CSS adaptables (Responsive Design):** Diseño oscuro (*Dark Racing Theme*) utilizando `CSS Grid`, variables CSS y `@media (max-width: 768px)` para adaptación a pantallas móviles.
* **Funcionalidad en JavaScript:** Simulador dinámico de telemetría que calcula en tiempo real la relación peso/potencia ($HP/kg$), tiempo estimado de aceleración de $0\text{ a }100\text{ km/h}$ según el mapa de motor (*Race, Sport, Rain*) y categorización automática de la motocicleta.

### 5.1. Comandos de creación de directorios, descarga de imágenes y edición:

```bash
# Crear carpeta para recursos gráficos
sudo mkdir -p /usr/share/nginx/html/img

# Descargar imágenes de motocicletas para el portal
sudo curl -L -o /usr/share/nginx/html/img/moto-hero.jpg "https://images.unsplash.com/photo-1558981403-c5f9899a28bc?auto=format&fit=crop&w=1200&q=80"
sudo curl -L -o /usr/share/nginx/html/img/moto-track.jpg "https://images.unsplash.com/photo-1568772585407-9361f9bf3a87?auto=format&fit=crop&w=800&q=80"

# Recrear y editar los archivos principales del sitio web
sudo rm -f /usr/share/nginx/html/index.html && sudo nano /usr/share/nginx/html/index.html
sudo rm -f /usr/share/nginx/html/style.css && sudo nano /usr/share/nginx/html/style.css
sudo rm -f /usr/share/nginx/html/script.js && sudo nano /usr/share/nginx/html/script.js
```

### 5.2. Estructura HTML (`/usr/share/nginx/html/index.html`):

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Apex Dynamics | Motociclismo de Alto Rendimiento</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&family=Orbitron:wght@600;800;900&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header class="navbar">
    <div class="nav-container">
      <div class="brand"><span class="brand-accent">APEX</span>CORSE</div>
      <nav class="nav-links">
        <a href="#ingenieria">Ingeniería</a>
        <a href="#telemetria">Telemetría JS</a>
        <a href="#especificaciones">Infraestructura</a>
      </nav>
    </div>
  </header>

  <section class="hero">
    <div class="hero-overlay"></div>
    <div class="hero-content">
      <span class="badge">PROYECTO EVALUATIVO // CLOUD IaaS</span>
      <h1>TELEMETRÍA & ALTO RENDIMIENTO</h1>
      <p>Plataforma de análisis dinámico y cálculo de relación peso/potencia para motocicletas de competición, desplegada sobre Oracle Cloud Infrastructure y Nginx.</p>
    </div>
  </section>

  <main class="container">
    <section id="ingenieria" class="card">
      <div class="card-header">
        <span class="sec-number">01</span>
        <h2>Ingeniería & Dinámica de Competición</h2>
      </div>
      <div class="grid-two">
        <div class="text-block">
          <p>El desarrollo de motocicletas de pista exige una convergencia estricta entre aerodinámica computacional (CFD), rigidez torsional del chasis y adquisición de telemetría en tiempo real.</p>
          <ul class="specs-list">
            <li><strong>Chasis:</strong> Doble viga perimetral en aleación de aluminio y subchasis en fibra de carbono.</li>
            <li><strong>Electrónica:</strong> ECU Marelli con control predictivo de tracción (DTC) y anti-wheelie.</li>
            <li><strong>Adquisición de Datos:</strong> Muestreo de sensores de suspensión y presión de frenado a 1000 Hz.</li>
          </ul>
        </div>
        <div class="img-block">
          <img src="img/moto-track.jpg" alt="Superbike en circuito" class="media-img">
        </div>
      </div>
    </section>

    <section id="telemetria" class="card">
      <div class="card-header">
        <span class="sec-number">02</span>
        <h2>Simulador de Potencia & Rendimiento (JavaScript)</h2>
      </div>
      <p class="section-desc">Ajusta los parámetros mecánicos para calcular la relación peso/potencia (HP/kg), aceleración estimada y categoría de circuito.</p>
      <div class="telemetry-panel">
        <div class="control-group">
          <label for="power-input">Potencia de Motor (HP):</label>
          <input type="number" id="power-input" value="160" min="50" max="250">
        </div>
        <div class="control-group">
          <label for="weight-input">Peso Conjunto Moto + Piloto (kg):</label>
          <input type="number" id="weight-input" value="230" min="120" max="350">
        </div>
        <div class="control-group">
          <label for="mode-select">Mapeo de Inyección (Ride Mode):</label>
          <select id="mode-select">
            <option value="1.0">Race / Track (100% Entrega)</option>
            <option value="0.92">Sport (92% Entrega lineal)</option>
            <option value="0.78">Rain / Wet (78% Curva suave)</option>
          </select>
        </div>
        <button id="calc-btn" class="btn-primary">CALCULAR TELEMETRÍA</button>
      </div>
      <div class="telemetry-results">
        <div class="result-box">
          <span class="metric-title">Relación Peso / Potencia</span>
          <span class="metric-val" id="ratio-output">0.70 HP/kg</span>
        </div>
        <div class="result-box">
          <span class="metric-title">0 - 100 km/h Estimado</span>
          <span class="metric-val" id="accel-output">3.12 s</span>
        </div>
        <div class="result-box">
          <span class="metric-title">Categoría de Desempeño</span>
          <span class="metric-val highlight" id="category-output">SuperSport</span>
        </div>
      </div>
    </section>

    <section id="especificaciones" class="card">
      <div class="card-header">
        <span class="sec-number">03</span>
        <h2>Arquitectura de Despliegue en Cloud</h2>
      </div>
      <div class="arch-grid">
        <div class="arch-item">
          <h4>Proveedor IaaS</h4>
          <p>Oracle Cloud Infrastructure (OCI Compute Instance)</p>
        </div>
        <div class="arch-item">
          <h4>Sistema Operativo</h4>
          <p>Oracle Linux Server (Kernel UEK / firewalld)</p>
        </div>
        <div class="arch-item">
          <h4>Servidor Web</h4>
          <p>Nginx Engine optimizado para entrega estática de baja latencia</p>
        </div>
        <div class="arch-item">
          <h4>Enrutamiento DNS</h4>
          <p>Resolución dinámica de nombres mediante DuckDNS</p>
        </div>
      </div>
    </section>
  </main>

  <footer class="footer">
    <p>Proyecto Evaluativo de Infraestructura Cloud &copy; 2026 | Despliegue en Servidores IaaS</p>
  </footer>
  <script src="script.js"></script>
</body>
</html>
```

### 5.3. Lógica en JavaScript (`/usr/share/nginx/html/script.js`):

```javascript
document.addEventListener('DOMContentLoaded', () => {
  const calcBtn = document.getElementById('calc-btn');
  const powerInput = document.getElementById('power-input');
  const weightInput = document.getElementById('weight-input');
  const modeSelect = document.getElementById('mode-select');

  const ratioOutput = document.getElementById('ratio-output');
  const accelOutput = document.getElementById('accel-output');
  const categoryOutput = document.getElementById('category-output');

  function calcularTelemetria() {
    const rawPower = parseFloat(powerInput.value) || 0;
    const weight = parseFloat(weightInput.value) || 1;
    const modeMultiplier = parseFloat(modeSelect.value) || 1.0;

    const effectivePower = rawPower * modeMultiplier;
    const ratio = (effectivePower / weight);
    ratioOutput.textContent = `${ratio.toFixed(2)} HP/kg`;

    let est0to100 = 6.5 / Math.sqrt(Math.max(ratio, 0.15));
    if (est0to100 < 2.3) est0to100 = 2.35;
    accelOutput.textContent = `${est0to100.toFixed(2)} s`;

    if (ratio >= 0.85) {
      categoryOutput.textContent = 'HyperSport / MotoGP';
      categoryOutput.style.color = '#ff2d2d';
    } else if (ratio >= 0.55) {
      categoryOutput.textContent = 'SuperSport 600-1000';
      categoryOutput.style.color = '#38bdf8';
    } else {
      categoryOutput.textContent = 'Sport-Touring / Naked';
      categoryOutput.style.color = '#4ade80';
    }
  }

  calcBtn.addEventListener('click', calcularTelemetria);
  calcularTelemetria();
});
```

### 5.4. Aplicación de Permisos y Contexto SELinux:

```bash
# Asignar permisos de lectura y ejecución al directorio web
sudo chmod -R 755 /usr/share/nginx/html

# Ajustar el contexto de seguridad de SELinux para permitir a Nginx servir el contenido
sudo chcon -Rt httpd_sys_content_t /usr/share/nginx/html/
```

### Evidencias del Paso 5:
![Edición en Nano](img/nano_edit.png)
![Comandos de Despliegue y SELinux](img/deploy_commands.png)

---

## 6. Configuración de Subdominio en DuckDNS y Comprobación Externa

1. Se ingresó a la plataforma [DuckDNS](https://www.duckdns.org/) y se registró un subdominio propio.
2. En el campo **current ip**, se asoció la dirección IPv4 pública de la máquina virtual en OCI: **`147.15.45.117`** y se guardaron los cambios mediante **update ip**.
3. Desde un navegador externo a la máquina virtual, se ingresó mediante HTTP al subdominio configurado, verificando:
   * La correcta visualización de las 3 secciones y las imágenes.
   * El funcionamiento interactivo de la calculadora de telemetría en JavaScript.
   * La adaptabilidad del diseño (Responsive CSS) en dispositivos móviles.

### Evidencias del Paso 6:
![Panel de Configuración de DuckDNS](img/duckdns_config.png)
![Comprobación en Navegador de Escritorio](img/web_desktop.png)
![Comprobación de Diseño Adaptable Móvil](img/web_mobile.png)