<div align="center">
  <img src="https://github.com/mrbuckwheet/Kometa-Config/assets/124317277/5323ca07-03fd-479a-b93d-681172b96290" alt="Kometa Config Logo">
  <h1>MrBuckwheet's Kometa Config</h1>
</div>

## 🚀 Installation

Extract the ENTIRE repository directory. Choose **Main**, **Minimalist**, or **Minimalist with UMTK** (only choose one) and place its contents into your Kometa config folder. 

1. Decide between the **Main**, **Minimalist**, or **Minimalist with UMTK** configuration.
2. Place all contents of your chosen folder into your **Kometa Config** folder. 
3. Ensure the inside of your Kometa config folder matches exactly what is inside your chosen folder.

> 📁 **UMTK Configuration Note:** If you are using the **Minimalist with UMTK** configuration, a `UMTK Config` folder is included in this repository with all the necessary configuration files to set up UMTK. Copy the contents of the `UMTK Config` folder into your `/{Container_Folder_PATH}/UMTK`.
>
> *⚠️You must add your Sonarr/Radarr API keys into the UMTK config.yml otherwise UMTK WILL NOT START⚠️*
> 
> You can use the provided Docker Compose setup below to run both Kometa and UMTK together.

> ⚠️ **IMPORTANT:** You **MUST** create a `UMTK` folder in your `/{Media_Folder_Path}` for the **Coming Soon** feature to work, as placeholders are used for the missing movies in Plex.

For more details on UMTK and its features, please visit the [**NetPlexFlix/UMTK GitHub Repository**](https://github.com/netplexflix/Upcoming-Movies-TV-Shows-for-Kometa).

> **Note:** All folders and files inside these directories (fonts, icons, additional `.yml` files, etc.) are needed for the configuration to work properly.

## Folder Directory Structure

```text
root/
├── Container folder/
│   ├── Kometa/
│   │   ├── assets/                 # Custom asset files
│   │   ├── Collection Posters/     # Artwork and posters for your collections
│   │   ├── missing/                # Missing media assets or logs
│   │   ├── overlays/               # Overlay configurations (e.g., resolution, audio tags)
│   │   ├── UMTK/
│   │   ├── config.yml              # Main Kometa configuration
│   │   └── Movie Collections.yml   # Collection definitions
│   ├── UMTK/
│   │   ├── config.yml
│   │   ├── localization.yml
│   │   └── tssk_config.yml
│   └── DV-Tagger/
└── Path/to/Media/
    ├── Movies/
    ├── UMTK/
    │   └── Coming Soon
    │       └── UMTK.mkv            # Sample media file for Coming Soon
    └── TV Shows/
```
---

## 🐳 Docker Compose (Kometa + UMTK + DV-Tagger)

### Setting Up Environment Variables
You must define the variables listed at the bottom of the Compose file. You have two recommended options:
* **Option 1 (.env file):** Create a `.env` file in the same directory as your `docker-compose.yml` and add the variables listed at the bottom of the compose there (e.g., `CONTAINER_FOLDER_PATH=/path/to/appdata`).
* **Option 2 (Portainer):** If you are deploying via Portainer, you can paste these variables directly into the "Environment variables" section of the stack editor.

```yaml
services:
  kometa:
    image: kometateam/kometa:latest
    container_name: kometa
    user: ${USER_ID}:${GROUP_ID}
    environment:
      - TZ=${TIME_ZONE}
      - KOMETA_TIMES=07:00,19:00
    volumes:
      - ${CONTAINER_FOLDER_PATH}/Kometa:/config
    restart: unless-stopped
    hostname: kometa
    networks: 
      - External

  umtk:
    image: netplexflix/umtk:latest
    container_name: umtk
    ports:
      - "2120:2120" # Web UI
    environment:
      - CRON=0 2 * * * # Run daily at 2am (used as initial default only)
      # - SCHEDULE_HOURS=24 # Alternative: run every X hours instead of a cron expression
      - DOCKER=true
      - TZ=${TIME_ZONE}
      - PUID=${USER_ID}
      - PGID=${GROUP_ID}
    #extra_hosts:
      #- "host.docker.internal:host-gateway" # Alternative way to access host
    volumes:
      # Configuration directory (required)
      - ${CONTAINER_FOLDER_PATH}/UMTK:/app/config

      # Video directory for placeholder method
      - ${MEDIA_FOLDER_PATH}/UMTK/Coming Soon:/video

      # Output directory for generated YAML files (required)
      - ${CONTAINER_FOLDER_PATH}/Kometa/UMTK:/app/kometa

      # UMTK root folders
      - ${MEDIA_FOLDER_PATH}/UMTK/Movies:/umtkmovies
      - ${MEDIA_FOLDER_PATH}/UMTK/TV Shows:/umtktv

    restart: unless-stopped
    hostname: umtk
    networks: 
      - External
      
  dv-tagger:
    image: mrbuckwheet/dolbyvision-tagger:latest
    container_name: dolbyvision-tagger
    ports:
      - "3636:3636"
    environment:
      - TZ=${TZ}
      - PUID=${USER_ID}
      - PGID=${GROUP_ID}
    volumes:
      - ${MEDIA_FOLDER_PATH}/Movies:/Movies                           # Path to Movies Library
      - ${CONTAINER_FOLDER_PATH}/DV-Tagger:/app/config                # Where your settings and logs will be saved
    restart: unless-stopped
    hostname: dv-tagger
    networks: 
      - External

networks:
  External:
    external: true

#### ENV VARIABLES USED ####
#CONTAINER_FOLDER_PATH=
#USER_ID=
#GROUP_ID=
#TIME_ZONE=
#MEDIA_FOLDER_PATH=
```
---

## ✨ Features

### New Minimalist Look with UMTK/TSSK
> ⚠️ **IMPORTANT:** For the minimalist templates to work correctly, you **MUST** use the [TRaSH naming convention](https://trash-guides.info/Radarr/Radarr-recommended-naming-scheme/#plex) for your file names.

### Dolby Vision Profile Collections
This config will automatically organize and create Dolby Vision collections categorized by their specific DV profiles.

To properly tag your media files with their respective Dolby Vision profiles so Kometa can read them, run the DV-Tagger at least once. [**Instructions Here**](https://github.com/mrbuckwheet/DolbyVision-Tagger#-previews).

![Dolby Vision Collections](Overlay%20Preview/DV%20Collections.png)

### Coming Soon Movies
The UMTK integration automatically pulls in metadata for movies that haven't been released or added to your main server yet, populating them into a completely separate library. This gives your users a clean sneak peek at upcoming media without cluttering your primary collections. For the best experience, you can pin this dedicated library directly to the Plex home screen for easy discovery.

![Coming Soon Preview](Overlay%20Preview/Coming%20Soon%20Preview.png)

> ⚠️ **IMPORTANT:** Create a New Library in Plex > Point the folder directory to `/path/to/your/Media/UMTK/Movies`. When Kometa runs once you will see Movies have been added to the new `Coming Soon` library. There will be a new Collection as well that will show on the Home screen. Match the Recommendations as shown and update your library order on Plex's Home page by sorting your Pinned Libraries. e.g., If you want Coming Soon to show before your Movie Library's Recently Added/Recently Released, pin then reorder the Coming Soon library to the top. Make sure to share the Coming Soon library with people you have invited to your server if you want them to see it too. 

<div align="center">
  <table border="0">
    <tr>
      <td>
        <img src="https://github.com/user-attachments/assets/b75e9910-c6f1-4bd4-a34f-b7d78558581c" height="300" alt="Screenshot 1">
      </td>
      <td>
        <img src="https://github.com/user-attachments/assets/f1c26a75-8181-4693-82f6-ae672fdb8973" height="300" alt="Screenshot 2">
      </td>
    </tr>
  </table>
</div>

### TV Show Status Updates (UMTK/TSSK)
Keep your TV Show library dynamically informed with essential production and broadcast milestones. This integration injects live status updates directly onto your series layouts, displaying next episode air dates, upcoming season premiere timelines, and season finale dates. It also tracks the network lifecycles of your shows, clearly detailing whether a series has been **Canceled**, **Ended**, or is **Returning** for another season.

![Minimalist TV-UMTK-TSSK](Overlay%20Preview/Minimalist%20TV-UMTK-TSSK.png)

### Pre-release Posters
This config automatically identifies **CAM** and **Telesync (TS)** versions of newly released movies in your library. When detected, Kometa applies a dedicated **Pre-Release Version** poster design in Plex. This visually alerts your users right away so they can easily distinguish between temporary, lower-quality theater rips and final retail quality releases.

![PreRelease Preview](Overlay%20Preview/PreRelease%20Preview.png)

### Dynamic Collection Builder
The dynamic collection builder in the `Movie Collections.yml` file will automatically make collections when 2 or more movies (this value can be customized) from the same collection are in your library. 

![Dynamic Collection Preview](Overlay%20Preview/Dynamic%20Collection%20Preview.png)

> 💡 **Recommendation:** It is recommended to disable "Minimum automatic collection size" under advanced library settings. You can also run the `python kometa.py --delete-collections` command if you have previous collections from Plex or manually added them before running Kometa. This prevents collection duplicates and gives you a clean start.

---

## 📸 Previews

<h3 align="center">Minimalist Theme</h3>

| Movie Overlay | TV Overlay | TV Overlays with UMTK |
| :---: | :---: | :---: |
| ![Minimalist Movie Overlay](Overlay%20Preview/Minimalist%20Movie%20Overlay.png) | ![Minimalist TV Overlay](Overlay%20Preview/Minimalist%20TV%20Overlay.png) | ![TV Overlays with UMTK](Overlay%20Preview/Minimalist%20TV-UMTK-TSSK.png) |

<br/>

<h3 align="center">Original Theme</h3>

| Movie Overlay | TV Overlay |
| :---: | :---: |
| ![Original Movies Overlay Preview](Overlay%20Preview/Original%20Movies%20Overlay%20Preview.JPG) | ![Original TV Overlay Preview](Overlay%20Preview/Original%20TV%20Overlay%20Preview.JPG) |

<br/>

<h3 align="center">Collections</h3>

| Movie Collections | TV Collection (Minimalist) |
| :---: | :---: |
| ![Movie Collections](Overlay%20Preview/Movie%20Collections.png) | ![Minimalist TV Collection](Overlay%20Preview/Minimalist%20TV%20Collection.png) |

---

## 👥 Credits & Acknowledgments

This utility suite leverages the incredible work of the following open-source media tool developers and projects:

* **[jmxd's Kometa config](https://github.com/jmxd/Kometa)**: The foundation for the New Minimalist Look (colored overlays, HD tags, top-left scores, bottom info/codecs).
* **[Upcoming-Movies-TV-Shows-for-Kometa (UMTK)](https://github.com/netplexflix/Upcoming-Movies-TV-Shows-for-Kometa)**: Developed by **netplexflix**. Used to create 'Coming Soon' collection and overlay `.yml` files for Kometa to display content being downloaded within a set number of days by Radarr/Sonarr.

---

<p align="center">
  <a href="https://www.buymeacoffee.com/mrbuckwheet" target="_blank">
    <img src="https://cdn.buymeacoffee.com/buttons/lato-black.png" alt="Buy Me A Coffee" style="height: 51px !important;width: 217px !important;">
  </a>
</p>
