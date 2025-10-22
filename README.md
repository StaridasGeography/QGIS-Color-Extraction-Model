# 🌈 RGB → HSV Color Clusters (QGIS Model)

A QGIS **Model Builder** tool that converts an **RGB raster** into **color clusters** based on **HSV (Hue–Saturation–Value)** color space.  
The model generates a **classified raster**, a **color palette map**, a **color table layer**, and an **Excel color table** — all ready for visualization or analysis.

---

## 🧠 What the model does

This model automates the process of **color-based segmentation** of RGB imagery using HSV conversion and clustering.

It performs:

1. **RGB → HSV conversion** (three raster layers: Hue, Saturation, Value).  
2. **K-Means color clustering** in HSV space.  
3. **Color reconstruction** from HSV to RGB, CMYK, and HEX for each cluster.  
4. **Creation of a color palette layer** placed around the map (Top / Right / Bottom / Left).  
5. **Export of color data** to both:
   - A **vector color table** (visible in your Layers Panel), and  
   - A **spreadsheet file** (`.xlsx` or `.ods`).  
6. **Optional CLR export** — a simple color look-up table for styling rasters.

---

## 📁 What’s included

- `RGB_to_HSV_Color_Clusters_v6.model3` → the QGIS Model (import directly into QGIS).  
- A workflow diagram image (`RGB_to_HSV_Color_Clusters_v6.png`) showing the full process visually.  
- `.qml` files for ready-made color symbology:
  - `Color Palette RGB.qml`
  - `Color Palette HSV.qml`
  - `Color Palette CMYK.qml`
  - `Color Palette HEX.qml`

---

## 🧩 Requirements

- **QGIS 3.4 or newer** (developed in QGIS 3.40.11, compatible with all recent versions).  
- The model uses built-in **QGIS**, **SAGA**, and **GRASS** processing tools — no extra plugins required.

---

## 🚀 Installation & setup (beginner-friendly)

### Step 1 — Add the model to QGIS

1. Open QGIS.  
2. Go to **Processing → Toolbox**.  
3. Click the **Models** tab (gear icon).  
4. Right-click → **Add Model** (or **Import Model**).  
5. Select the file:  
   ```
   RGB_to_HSV_Color_Clusters_v6.model3
   ```
6. The model will now appear in your **Processing Toolbox**, usually under **Models → RGB_to_HSV_Color_Clusters_v6**.

---

### Step 2 — Prepare your data

- Load an **RGB raster** (3-band image, e.g., orthophoto or render).  
- Set your **map canvas** to the area you want to analyze — the model uses it for **Extent**.

---

### Step 3 — Run the model

Open **Processing → Toolbox → Models → RGB_to_HSV_Color_Clusters_v6**, then set the following parameters:

| Parameter | Description |
|------------|-------------|
| **Map** | Your input RGB raster layer. |
| **Extent** | Defines the area to process (optional — uses layer or map extent). |
| **Number of Colors (Clusters)** | Number of color groups (default **10**). |
| **Color Palette (map) Placement** | Where to place the color palette (Top / Right / Bottom / Left). |
| **CRS** | Coordinate system for outputs (default **EPSG:3857**). |
| **Color Palette (map)** | Output vector showing swatches of each cluster color. |
| **Color Table (Layer)** | Output attribute table with detailed color values (appears in Layers Panel). |
| **Color Table XLSX** | Spreadsheet export of the color table. |
| **HSV Raster Layer (Clusters)** | Output raster with cluster IDs per pixel. |
| **HSV Raster Layer CLR** | Optional color look-up file for quick styling. |

Then click **Run** 🟢.

---

## 📦 Model outputs

| Output | Description |
|---------|-------------|
| 🗺️ **HSV Raster Layer (Clusters)** | Raster where each pixel belongs to a color cluster (1…N). |
| 🎨 **Color Palette (map)** | Vector of rectangles showing each cluster’s color, positioned around the map extent. |
| 📋 **Color Table (Layer)** | Vector table with cluster statistics and color data (automatically added to the Layers Panel). |
| 📘 **Color Table XLSX** | Spreadsheet file (`.xlsx` or `.ods`) with the same color information. |
| 🧾 **HSV Raster Layer CLR** | Optional simple text file mapping cluster IDs to RGB values for styling. |

---

## 🎨 Apply QGIS Style Files (Optional)

You can use the provided `.qml` style files to colorize your **Color Palette (map)** layer in different color models:

| File | Description |
|------|--------------|
| `Color Palette RGB.qml` | Styles palette polygons using RGB values. |
| `Color Palette HSV.qml` | Styles palette polygons using HSV colors. |
| `Color Palette CMYK.qml` | Styles palette polygons using CMYK approximation. |
| `Color Palette HEX.qml` | Styles palette polygons using web HEX codes. |

**How to apply a style:**
1. In the **Layers Panel**, right-click your *Color Palette (map)* layer.  
2. Choose **Properties → Symbology**.  
3. Click the **Style** button (bottom-left) → **Load Style…**  
4. Select one of the provided `.qml` files and click **OK**.  
   → The layer will instantly adopt the selected color scheme.

---

## 🖌️ Style the Cluster Raster Layer using the CLR file

The model exports a `.clr` file that can automatically colorize your **HSV Raster Layer (Clusters)**.

**How to apply the CLR style:**

1. In QGIS, right-click the **HSV Raster Layer (Clusters)** in your **Layers Panel**.  
2. Select **Properties → Symbology**.  
3. Change **Render type** to **Paletted / Unique values**.  
4. At the bottom, click **Load Color Map from File...**  
5. Choose the generated `.clr` file (e.g., `HSV Raster Layer CLR.csv`).  
6. The raster will automatically adopt the correct cluster colors.

> 💡 Each line in the `.clr` file follows this format:
> ```
> id R G B 255 id
> ```
> where `id` is the cluster number and `R G B` are the red, green, and blue values for that cluster.

---

## 🧬 Theory & logic

### 1. RGB → HSV conversion
The model first computes **Hue**, **Saturation**, and **Value** rasters from the input RGB bands using standard formulas:

- **Value (V)**:  
  `V = 100 * max(R, G, B) / 255`
- **Saturation (S)**:  
  `S = 100 * (max(R, G, B) - min(R, G, B)) / max(R, G, B)`  
  (if max = 0, then S = 0)
- **Hue (H)**:  
  Piecewise function based on which channel is dominant, scaled to 0–360°.

These conversions are implemented with **QGIS Raster Calculator** expressions.

> 💡 HSV color space separates brightness (Value) from chroma (Hue, Saturation), making it more perceptually intuitive for clustering.

---

### 2. K-Means clustering
Once the H, S, and V rasters are computed, they’re fed into **SAGA’s “K-Means Clustering for Rasters”** algorithm:

- Clusters colors in 3D HSV space.  
- Uses **periodical initialization** and **hillclimbing** optimization.  
- Normalizes each band before clustering.  
- Output:  
  - A **clustered raster**,  
  - A **cluster statistics table** (mean, std. dev., cluster ID, etc.).

Each cluster represents a **dominant color region** in your image.

---

### 3. Color reconstruction (HSV → RGB/CMYK/HEX)
For each cluster, the model calculates:

- **RGB values** converted back from H, S, V using the standard piecewise HSV→RGB formula.  
- **CMYK values** derived from RGB.  
- **HEX color code** (`#RRGGBB`) for easy use in design tools or web color palettes.

These fields are computed using advanced **QGIS field calculator expressions** with nested `with_variable()` functions.

---

### 4. Palette generation
The **Color Palette (map)** is built with the **GRASS `v.mkgrid`** algorithm:
- Creates a series of squares arranged in a line (horizontal or vertical) based on the chosen placement.  
- Each square corresponds to one color cluster.  
- Automatically positioned just outside the map extent (so it doesn’t overlap your raster).

---

### 5. Color table exports
- The **Color Table (Layer)** is a standard vector table that appears automatically in your **Layers Panel**.  
  You can open its attribute table to inspect HSV, RGB, CMYK, and HEX values.
- The **Color Table XLSX** is exported using the native QGIS *Export to Spreadsheet* algorithm for external use (Excel, LibreOffice, etc.).  
- The **CLR file** contains simple text entries in the format:  
  ```
  id R G B 255 id
  ```

---

## 📖 License
CC BY-NC-ND 4.0

---

## 🙌 Credits
- Developed with **QGIS Model Builder**.  
- Uses **SAGA** and **GRASS GIS** algorithms through QGIS Processing Framework.  
- Model name: `RGB_to_HSV_Color_Clusters_v6`.

---

## 💡 Future improvements
- Optional smoothing filter before clustering.  
- LAB/LCH color space option.  
- Auto-labeling of color palette swatches.
