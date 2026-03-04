# AR Navigation

A browser-based Augmented Reality navigation app built with **A-Frame** and **AR.js**.  
Point your device camera at the printed marker and see a red directional arrow and navigation text overlaid in 3D.

---

## How It Works

| Layer | Role |
|-------|------|
| **A-Frame** | Declarative 3D/WebXR framework. Defines the scene, camera, and 3D objects using HTML-like tags. |
| **AR.js** | Adds camera access and real-time marker tracking to A-Frame. When the marker is in view, AR.js computes its pose and A-Frame renders children anchored to it. |
| **pattern-marker.patt** | Encodes the black-and-white bitmap of the printed marker. AR.js loads this file to know what image to look for. |

### Rendering pipeline (simplified)

```
Device Camera → AR.js (marker detection) → A-Frame (3D overlay) → Screen
```

1. `<a-scene arjs>` opens the device camera and starts the AR.js tracking loop.
2. Every frame, AR.js scans the camera feed for the bitmap described in `pattern-marker.patt`.
3. When the marker is found, AR.js computes a 6-DOF pose (position + rotation in 3D space).
4. A-Frame renders the cone and text as children of `<a-marker>`, anchored to that pose.

---

## Files

```
ar-navigation/
├── index.html           # Single-page AR app (all code lives here)
└── pattern-marker.patt  # AR.js pattern descriptor for the printed marker
```

---

## Running Locally

> **HTTPS required** – browsers only grant camera access to secure contexts.

### Option 1 – Python simple HTTPS server (quickest)

```bash
# Generate a self-signed certificate (one-time)
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=localhost" -addext "subjectAltName=DNS:localhost"

# Serve the current directory over HTTPS
python3 -c "
import ssl, http.server
ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
ctx.load_cert_chain('cert.pem', 'key.pem')
httpd = http.server.HTTPServer(('0.0.0.0', 8443), http.server.SimpleHTTPRequestHandler)
httpd.socket = ctx.wrap_socket(httpd.socket, server_side=True)
httpd.serve_forever()
"
```

Then open `https://localhost:8443` in your browser (accept the self-signed certificate warning).

### Option 2 – VS Code Live Server + ngrok

1. Install the **Live Server** VS Code extension and start it (port 5500 by default).
2. Install [ngrok](https://ngrok.com/) and run `ngrok http 5500` to get a public HTTPS URL.
3. Open the ngrok URL on your mobile device.

---

## Printing the Marker

Print `pattern-marker.patt` indirectly – the `.patt` file cannot be printed directly—it is a numeric descriptor, not an image.  
To get the printable image:

1. Visit the [AR.js Marker Generator](https://jeromeetienne.github.io/AR.js/three.js/examples/marker-training/examples/generator.html).
2. Upload your source image and download both the marker PNG and the `.patt` file.
3. Replace `pattern-marker.patt` with the new `.patt` file and update the `url` attribute in `index.html` if needed.
4. Print the PNG image in a high-contrast black-and-white, at least 8 cm × 8 cm.

---

## Customising the Navigation Overlay

All visual properties are set directly in `index.html`:

| What to change | Where to find it | Attribute to edit |
|----------------|-----------------|-------------------|
| Arrow colour | `<a-entity material="color: red …">` | `material="color: …"` |
| Arrow size | `<a-entity … scale="0.5 0.5 0.5">` | `scale` |
| Direction text | `<a-text value="Go Forward" …>` | `value` |
| Text colour | `<a-text … color="black" …>` | `color` |
| Text position | `<a-text … position="-0.3 0.3 0" …>` | `position` |

---

## Browser & Device Compatibility

| Browser | Desktop | Android | iOS |
|---------|---------|---------|-----|
| Chrome | ✅ | ✅ | – |
| Firefox | ✅ | ✅ | – |
| Safari | – | – | ✅ (iOS 14.3+) |

A camera with autofocus gives the best tracking results.

---

## Dependencies

- [A-Frame 1.2.0](https://aframe.io/) – MIT licence
- [AR.js](https://ar-js-org.github.io/AR.js-Docs/) – MIT licence
