```markdown

\# Earth Data Pipeline \& Unity Integration — Save locations and how Play starts everything



Project structure (relevant parts)

Assets/

&nbsp; Scripts/

&nbsp;   TextureReloader.cs                <- Unity runtime component (drop-in)

&nbsp; Editor/

&nbsp;   StartEarthPipelineOnPlay.cs       <- Editor helper that starts/stops Python pipeline when Play is pressed

&nbsp; StreamingAssets/

&nbsp;   EarthData/

&nbsp;     raw/

&nbsp;       Aurora/

&nbsp;       Cloud/

&nbsp;       Current/

&nbsp;       Wind/

&nbsp;     processed/

&nbsp;       Aurora/

&nbsp;         aurora.png

&nbsp;       Cloud/

&nbsp;         cloud.png

&nbsp;       Current/

&nbsp;         current.png

&nbsp;       Wind/

&nbsp;         wind.png

&nbsp;     pipeline/

&nbsp;       earth\_data\_pipeline.py        <- Python pipeline (run by Editor helper on Play)

&nbsp;       copernicus\_helper.py          <- helper used by the pipeline



What to put where

\- Place TextureReloader.cs in Assets/Scripts/.

\- Place StartEarthPipelineOnPlay.cs in Assets/Editor/.

\- Place the python scripts in Assets/StreamingAssets/EarthData/pipeline/.

&nbsp; (You can also run the Python pipeline from anywhere; placing it inside StreamingAssets keeps it with the project.)



Python dependencies (install on the machine that will run the pipeline)

\- pip install requests pillow numpy

\- Optional/needed for GRIB/NetCDF:

&nbsp; pip install xarray netCDF4 cfgrib

&nbsp; and install ecCodes on the system (cfgrib dependency; apt/brew/homebrew install eccodes)

\- Optional for Copernicus listing parsing:

&nbsp; pip install beautifulsoup4



How it is started when you press Play in the Editor

\- The Editor script StartEarthPipelineOnPlay.cs detects Play Mode entry and attempts to start the Python pipeline as a subprocess:

&nbsp; - It uses "python" or "python3" from PATH by default.

&nbsp; - It runs the pipeline script located at Assets/StreamingAssets/EarthData/pipeline/earth\_data\_pipeline.py (you can change the path by editing the constants in the Editor script).

&nbsp; - On exit from Play Mode it kills the background process.

\- The TextureReloader component (attach to a GameObject in the scene) polls the processed PNGs (preferentially Application.persistentDataPath/EarthData/processed, then Application.streamingAssetsPath/EarthData/processed) and injects them into Visual Effect(s) using VisualEffect.SetTexture. Set the VisualEffect and property names in the inspector.



Development \& production differences

\- Editor: Python pipeline writes into Assets/StreamingAssets/EarthData/processed/ and Unity reads the PNGs. Editor script starts/stops pipeline automatically when you press Play (good for iterating).

\- Build: StreamingAssets may be read-only inside built players. For builds either:

&nbsp; - run pipeline on the same machine and configure it to write into persistentDataPath (or have a copy step), or

&nbsp; - configure TextureReloader to pull from a shared folder (persistentDataPath is preferred).



Testing steps (first run)

1\. Save all files per above paths.

2\. Install Python dependencies.

3\. Run a single pipeline cycle manually to ensure PNGs are produced:

&nbsp;  python Assets/StreamingAssets/EarthData/pipeline/earth\_data\_pipeline.py --once --project-root /path/to/your/unity/project

4\. Open Unity, add TextureReloader component to a GameObject and configure target VisualEffect(s) \& property names.

5\. Press Play. The Editor helper will start the Python pipeline automatically and TextureReloader will pick up produced PNGs and set them into your VFX Graph.



If you want, I can:

\- Produce a small Editor UI to configure the pipeline python path from Unity Editor Preferences.

\- Provide a systemd service / Windows service example for running the pipeline automatically outside the Editor.

\- Add a GUI debug window in Unity to show the currently loaded files and last update times.



```

