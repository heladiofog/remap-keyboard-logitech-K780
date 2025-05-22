# Remapeo de Teclas en macOS para Teclado Logitech K780 (Distribución LATAM)

Este tutorial documenta el proceso para solucionar un problema de mapeo incorrecto de teclas al usar un teclado **Logitech K780** con **distribución latinoamericana (Español de México)** en una **MacBook Pro con chip Apple Silicon (M*)**, utilizando la herramienta **Karabiner-Elements**.

## 🚫 Problema

Al conectar el teclado Logitech K780 por Bluetooth a una MacBook Pro:

* macOS **no reconoce correctamente las teclas de signos "<" y ">"**, ubicadas a la izquierda de la tecla `Z`.
* La tecla a la izquierda del número `1` sí funciona correctamente generando los caracteres `|` y `°`, pero **deja de hacerlo si no se habilita la regla para remapear la tecla a la izquierda de la `Z`**.

Mientras que el teclado interno de la MacBook funciona con la distribución esperada, el K780 presenta un mapeo incorrecto en la tecla de menor/mayor que (`<` y `>`).

## 📄 Solución

Usaremos **Karabiner-Elements**, una poderosa herramienta para remapear teclas a nivel de sistema.

### ✅ Requisitos

* Tener instalado [Karabiner-Elements](https://karabiner-elements.pqrs.org/)
* Teclado Logitech K780 con distribución latinoamericana (Español de México)
* macOS con permisos suficientes para controlar el sistema (habilitar en "Security & Privacy")

---

## 1. Identificar los códigos de las teclas afectadas

Usamos la herramienta integrada de Karabiner llamada **EventViewer**:

1. Abre `Karabiner-Elements`
2. Ve a la pestaña **EventViewer**
3. Presiona la tecla **a la izquierda de Z**

   * Se reporta como: `grave_accent_and_tilde`
4. Presiona la tecla **a la izquierda del 1**

   * Se reporta como: `non_us_backslash`

Estas teclas **deberían generar los siguientes caracteres**:

| Tecla física      | Caracter sin Shift | Con Shift |     |
| ----------------- | ------------------ | --------- | --- |
| Izquierda de `Z`  | `<`                | `>`       |     |
| Izquierda del `1` | \|                 | \°        | `°` |

**Valida que las teclas requieren este tipo de mapeos, a veces solo con instalar la herramienta se afecta el mapeo del teclado.**
---

## 2. Crear reglas personalizadas en Karabiner

Creamos un archivo JSON que define estas remapificaciones usando `shell_command` para emitir caracteres directamente.

### Ruta del archivo:

```bash
~/.config/karabiner/assets/complex_modifications/remap_k780_latam.json
```

### Contenido del archivo:

```json
{
  "title": "Fix Logitech K780 LATAM keymap (< > | °)",
  "rules": [
    {
      "description": "Map grave_accent_and_tilde to < and >",
      "manipulators": [
        {
          "type": "basic",
          "from": {
            "key_code": "grave_accent_and_tilde",
            "modifiers": {
              "optional": ["any"]
            }
          },
          "to": [
            {
              "shell_command": "osascript -e 'tell application \"System Events\" to keystroke \"<\"'"
            }
          ]
        },
        {
          "type": "basic",
          "from": {
            "key_code": "grave_accent_and_tilde",
            "modifiers": {
              "mandatory": ["shift"]
            }
          },
          "to": [
            {
              "shell_command": "osascript -e 'tell application \"System Events\" to keystroke \">\"'"
            }
          ]
        }
      ]
    }
  ]
}
```

## Alternativamente (OPCIÓN B de archivos)
Se pueden tener dos reglas separadas para que cada tecla conserve un mapeo independiente si es que el teclado lo permite. Contenido de los archivo:

- Crear el archivo para mapear tecla a la izquierda de la 'z', de (|°) a (<>):
`vi ~/.config/karabiner/assets/complex_modifications/remap_grave_to_less_greater.json`

```json
{
  "title": "Map grave_accent_and_tilde to < and >",
  "rules": [
    {
      "description": "Change grave_accent_and_tilde to < (and > with Shift)",
      "manipulators": [
        {
          "type": "basic",
          "from": {
            "key_code": "grave_accent_and_tilde",
            "modifiers": {
              "optional": ["any"]
            }
          },
          "to": [
            {
              "shell_command": "osascript -e 'tell application \"System Events\" to keystroke \"<\"'"
            }
          ]
        },
        {
          "type": "basic",
          "from": {
            "key_code": "grave_accent_and_tilde",
            "modifiers": {
              "mandatory": ["shift"]
            }
          },
          "to": [
            {
              "shell_command": "osascript -e 'tell application \"System Events\" to keystroke \">\"'"
            }
          ]
        }
      ]
    }
  ]
}
```

- Crear el archivo para mapear tecla a la izquierda del '1' (superior a la izq.), para que permanezca igual:
`vi ~/.config/karabiner/assets/complex_modifications/remap_grave_to_less_greater.json`

```json
{
  "title": "Fix < and > on Logitech Latin Keyboard",
  "rules": [
    {
      "description": "Map non_us_backslash (|/°) to < and > on Logitech K780",
      "manipulators": [
        {
          "type": "basic",
          "from": {
            "key_code": "non_us_backslash",
            "modifiers": {
              "optional": ["any"]
            }
          },
          "to": [
            {
              "key_code": "grave_accent_and_tilde"
            }
          ]
        },
        {
          "type": "basic",
          "from": {
            "key_code": "non_us_backslash",
            "modifiers": {
              "mandatory": ["shift"]
            }
          },
          "to": [
            {
              "key_code": "grave_accent_and_tilde",
              "modifiers": ["shift"]
            }
          ]
        }
      ]
    }
  ]
}
```

---

## 3. Activar la regla en Karabiner

1. Abre **Karabiner-Elements > Complex Modifications**
2. Haz clic en **"Add Rule"**
3. Busca la regla: `Fix Logitech K780 LATAM keymap (< > | °)`
4. Haz clic en **"Enable"**

---

## 🧠 Explicación técnica

* macOS a veces asigna el **mismo key\_code (`grave_accent_and_tilde`)** a dos teclas distintas en teclados internacionales, lo que puede causar conflictos si solo una de ellas se remapea.
* En el caso del K780 con layout LATAM:

  * La tecla de menor/mayor (`<` y `>`) usa `grave_accent_and_tilde`
  * La tecla de barra vertical y grado (`|` y `°`) usa `non_us_backslash` y **funciona correctamente por defecto**
* Sin embargo, si **no se habilita la regla que remapea `grave_accent_and_tilde`**, macOS interpreta erróneamente ambas teclas como iguales, causando que ambas generen `<` y `>`.

Por lo tanto, **aunque la tecla `|` y `°` funcione por defecto, requiere que la regla de `<` y `>` esté activa para no ser afectada por el conflicto de key\_codes**.

---

## 🔧 Reiniciar si es necesario

Si no ves cambios:

* Ve a la pestaña **"Misc"** de Karabiner y presiona **"Restart Karabiner-Elements"**, o
* Cierra sesión y vuelve a iniciar en macOS

---

## 🚀 Resultado esperado

| Tecla física      | Tecla generada |        |
| ----------------- | -------------- | ------ |
| Izquierda de `Z`  | `<` y `>`      |        |
| Izquierda del `1` | \`             | `y`°\` |

---

## 📆 Historial

* Mayo 2025: Solución documentada por Heladio AF en base a un caso real con MacBook Pro M* + teclado Logitech K780 LATAM
** Ayuda de chatGPT para resolver esta situación.

---

## 📢 Contribuciones

Si tienes otros teclados o distribuciones que requieran ajustes similares, puedes abrir un PR o issue en este repositorio.

