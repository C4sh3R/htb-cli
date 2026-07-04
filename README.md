# htb — Cliente de terminal para HackTheBox

> CLI minimalista en Bash para HackTheBox usando la **API oficial** (v4, con v5 para el envío de flags). Diseñado para ir rápido: spawnea máquinas, coge la IP, manda flags — sin salir de la terminal.

```
htb spawn Connected      →  spawnea la máquina, espera la IP y la copia al portapapeles
htb own <hash>           →  sube la flag a la máquina activa automáticamente
htb season               →  lista las máquinas de la season con estado de bloods
```

---

## Características

- **Solo necesita `curl` y `jq`** — sin Python, sin npm, sin dependencias raras
- Resuelve nombres de máquinas a IDs automáticamente (busca en season → lista general → búsqueda libre)
- Después del spawn, espera la IP y la copia al portapapeles (`xclip`)
- Detecta la máquina activa automáticamente en `own`, `stop`, `reset` e `ip`
- Salida con colores y prefijos claros `[+]` / `[!]` / `[*]`
- El token se guarda una vez en `~/.config/htb/token` o se pasa por variable de entorno
- Gestión de VPN integrada: descarga la config, conecta, comprueba `tun0`

---

## Requisitos

| Herramienta | Para qué |
|-------------|----------|
| `bash` ≥ 4 | Shell |
| `curl` | Llamadas a la API |
| `jq` | Parseo de JSON |
| `xclip` | Copiar IP al portapapeles tras spawn (opcional) |
| `openvpn` | VPN con `htb connect` (opcional) |

---

## Instalación

```bash
# 1. Clonar
git clone https://github.com/C4sh3R/htb-cli.git
cd htb-cli

# 2. Instalar
chmod +x htb
cp htb ~/.local/bin/htb        # o: sudo cp htb /usr/local/bin/htb

# 3. Guardar el token de la API
mkdir -p ~/.config/htb
echo "TU_TOKEN_AQUI" > ~/.config/htb/token
chmod 600 ~/.config/htb/token
```

El token lo sacas de **HackTheBox → Account Settings → API Token**.

### Alternativa: variable de entorno

```bash
export HTB_TOKEN="eyJ0eXAiOiJKV1..."
```

### VPN por defecto

> ⚠️ **Importante:** la ruta de la VPN es personal — depende de dónde tengas guardado tu `.ovpn`. La variable `VPN_FILE` al principio del script apunta a la ruta del autor, así que **cámbiala por la tuya** (o usa `HTB_VPN_FILE`), o pásale el archivo directamente a `htb connect /ruta/a/tu.ovpn`.

```bash
export HTB_VPN_FILE="/ruta/a/tu/machines.ovpn"
```

O edita la variable `VPN_FILE` al principio del script.

---

## Uso

```
htb <comando> [argumentos]
```

### Referencia rápida

| Comando | Descripción |
|---------|-------------|
| `htb whoami` | Muestra tu usuario y estado VIP |
| `htb list [N]` | Últimas N máquinas (por defecto 15), más recientes primero |
| `htb latest` | La máquina más reciente (la release de hoy) |
| `htb season` | Máquinas de la season activa con estado de bloods |
| `htb info <nombre\|id>` | Perfil completo de una máquina |
| `htb active` | Máquina spawneada ahora mismo + IP |
| `htb spawn <nombre\|id>` | Spawnea una máquina y espera la IP |
| `htb ip [nombre\|id]` | Espera e imprime la IP de la máquina activa |
| `htb stop [nombre\|id]` | Termina la máquina activa (o la que indiques) |
| `htb reset [nombre\|id]` | Resetea la máquina activa (o la que indiques) |
| `htb own <flag> [dif]` | Sube una flag a la máquina **activa** |
| `htb own <nombre\|id> <flag> [dif]` | Sube una flag a una máquina concreta |
| `htb vpn [archivo]` | Descarga tu config VPN desde la API |
| `htb connect [archivo]` | Conecta la VPN y verifica `tun0` |
| `htb disconnect` | Desconecta la VPN (mata `openvpn`, espera a que `tun0` baje) |
| `htb vpnstatus` | Comprueba si `tun0` está activo |

---

## Comandos en detalle

### `htb whoami`

Comprueba que el token funciona y muestra tu cuenta.

```
$ htb whoami
Usuario: C4sh3R  (id 12345)
VIP: true  Server: eu-vip-2
```

---

### `htb list [N]`

Las N máquinas más recientes. Muestra si ya tienes user (`U:1`) o root (`R:1`).

```
$ htb list 5
906  Connected  Linux   Easy    2025-11-30  U:1 R:0
905  VariaType  Linux   Medium  2025-11-23  U:0 R:0
904  Pupilpath  Windows Hard    2025-11-16  U:1 R:1
```

---

### `htb latest`

La máquina más reciente de todas. Ideal para ver qué ha salido hoy.

```
$ htb latest
Mas reciente:
  ID:         906
  Nombre:     Connected
  OS:         Linux
  Dificultad: Easy
  Release:    2025-11-30T20:00:00.000000Z
  Activa:     1
```

---

### `htb season`

Todas las máquinas de la season activa con fecha de release y estado de bloods.

```
$ htb season
906  Connected  Linux  Easy    rel:2025-11-30  released:true  Ublood:false  Rblood:false
905  VariaType  Linux  Medium  rel:2025-11-23  released:true  Ublood:true   Rblood:false
```

---

### `htb spawn <nombre|id>`

Spawnea por nombre o por ID numérico. El script resuelve el nombre en este orden:

1. Lista de máquinas de la season (las nuevas releases están aquí)
2. Lista paginada general (top 100 recientes)
3. Endpoint de búsqueda libre de HTB

Tras el spawn, hace polling cada 2 segundos hasta que la IP está asignada, la imprime y la copia al portapapeles.

```
$ htb spawn Connected
[*] Spawneando maquina id 906 ...
[+] Machine deployed
[*] Esperando IP...
[+] IP: 10.129.9.213
    (copiada al portapapeles)
```

---

### `htb ip`

Espera hasta 80 segundos a que la máquina activa tenga IP. Útil si spawneaste en otra sesión.

```
$ htb ip
[+] IP: 10.129.9.213
    (copiada al portapapeles)
```

---

### `htb active`

Muestra qué máquina tienes spawneada ahora mismo.

```
$ htb active
Activa: Connected (id 906)
IP: 10.129.9.213
Expira: 2025-12-01 08:00:00
```

---

### `htb own <flag> [dificultad]`

Sube una flag a la máquina activa. La dificultad es un número del 1 al 10 (por defecto 5).

```bash
# Flag a la máquina activa (auto-detectada)
htb own 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d

# Con dificultad explícita
htb own 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d 3

# A una máquina específica por nombre
htb own Connected 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d
```

Respuesta si es correcta:

```
[+] Congratulations! You have got the user flag on Connected.
```

---

### `htb reset`

Resetea la máquina activa. Muy útil cuando algo se queda pillado durante la explotación.

```
$ htb reset
[+] The machine has been scheduled for reset.
```

---

### `htb stop`

Termina la máquina y libera el slot de VPN.

```
$ htb stop
[+] Machine stopped.
```

---

### `htb info <nombre|id>`

Perfil completo de una máquina: OS, dificultad, puntos, IP, si la tienes pwneada...

```
$ htb info Connected
{
  "id": 906,
  "name": "Connected",
  "os": "Linux",
  "difficultyText": "Easy",
  "ip": "10.129.9.213",
  "points": 20,
  "authUserInUserOwns": true,
  "authUserInRootOwns": false
}
```

---

### `htb vpn [archivo]`

Descarga tu configuración VPN actual desde la API (por defecto la guarda como `htb.ovpn`).

```bash
htb vpn
htb vpn ~/vpn/eu-vip.ovpn
```

---

### `htb connect [archivo.ovpn]`

Conecta la VPN con `openvpn --daemon` y espera hasta 15 segundos a que `tun0` suba.

> La ruta por defecto depende de dónde tengas tu `.ovpn`. Si no pasas archivo, usa `VPN_FILE`/`HTB_VPN_FILE` — asegúrate de haberla puesto a tu ruta (ver [VPN por defecto](#vpn-por-defecto)). También puedes pasar el archivo directamente:

```
$ htb connect ~/vpn/mi-config.ovpn
[*] Conectando VPN con /ruta/machines.ovpn (sudo)...
[+] VPN arriba: 10.10.14.5/23
```

Si guardas tu contraseña de sudo en `~/.config/htb/sudo.pass`, la conexión es completamente no interactiva:

```bash
echo "tu_password_sudo" > ~/.config/htb/sudo.pass
chmod 600 ~/.config/htb/sudo.pass
```

---

### `htb disconnect`

Corta la VPN: mata el proceso `openvpn` y espera hasta 15 segundos a que `tun0` desaparezca. Usa `~/.config/htb/sudo.pass` si existe para no pedir contraseña.

```
$ htb disconnect
[*] Desconectando VPN (sudo)...
[+] VPN desconectada (tun0 abajo).
```

---

### `htb vpnstatus`

Comprueba rápidamente si `tun0` está activo.

```
$ htb vpnstatus
[+] VPN conectada: 10.10.14.5/23
```

---

## Workflow típico — First Blood

```bash
# 1. Conectar VPN (una vez por sesión)
htb connect

# 2. Ver qué ha salido
htb latest
htb season

# 3. Spawnear
htb spawn Connected

# 4. ... hackear ...

# 5. Subir flags
htb own <hash_user>
htb own <hash_root>

# 6. Limpiar
htb stop
htb disconnect   # cortar la VPN al terminar
```

---

## Ejemplos de uso frecuentes

```bash
# Ver si tienes la VPN activa
htb vpnstatus

# Spawnear por nombre o por ID
htb spawn 906
htb spawn Connected

# Ver la máquina que tienes ahora mismo
htb active

# Resetear si la máquina se ha quedado pillada
htb reset

# Subir flag con dificultad baja (era fácil)
htb own abc123... 2

# Subir flag a una máquina concreta sin que sea la activa
htb own VariaType def456... 7

# Listar las últimas 30 máquinas
htb list 30

# Buscar info de cualquier máquina
htb info "Lame"
htb info 1
```

---

## Seguridad del token

El token se lee en este orden:

1. Variable de entorno `$HTB_TOKEN`
2. Fichero `~/.config/htb/token` (o `$HTB_TOKEN_FILE`)

Asegúrate de que el fichero solo lo puede leer tu usuario:

```bash
chmod 600 ~/.config/htb/token
```

---

## Variables de configuración

| Variable | Por defecto | Descripción |
|----------|-------------|-------------|
| `HTB_TOKEN` | — | Token de la API (override por env) |
| `HTB_TOKEN_FILE` | `~/.config/htb/token` | Ruta al fichero de token |
| `HTB_VPN_FILE` | *(ruta hardcodeada)* | Config VPN por defecto para `htb connect` |

---

## Licencia

MIT — úsalo como quieras.
