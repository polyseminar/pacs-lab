# PACS lab

A free PACS on your own computer, for learning. It runs [Orthanc](https://www.orthanc-server.com/), an open-source DICOM server, with the OHIF and Stone web viewers, and loads one public, de-identified CT study.

This lab goes with the Polyseminar video "Build a free PACS on your laptop in 10 minutes".

> **Lab use only.** Load only the public test data named below. Never load real patient data, including your own scans.

## What you need

- **Docker**, with at least 8 GB of RAM in the computer:

  | System | What to install |
  | :--- | :--- |
  | Windows 11 | [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) with the WSL 2 backend (the default). The first setup of WSL 2 needs administrator rights once and may restart the computer. |
  | macOS | [Docker Desktop](https://docs.docker.com/desktop/setup/install/mac-install/) for Apple silicon or Intel |
  | Linux | [Docker Engine](https://docs.docker.com/engine/install/) with the Compose plugin |

  Windows 10 and Windows Home editions should work but are not tested.

- **About 4 GB of free disk space.** The Orthanc image uses about 2.6 GB and the study about 450 MB (the files plus a ZIP copy).

Docker Desktop is free for personal use, education, and small businesses (fewer than 250 employees and less than $10 million in annual revenue). Larger organizations and government entities need a paid subscription ([Docker Desktop license terms](https://docs.docker.com/subscription-billing/desktop-license/)). On a work computer, ask your IT team before you install it.

## Open a terminal in the lab folder

Download the lab files (**Code > Download ZIP** on GitHub) and extract them, or clone the repository. Then open a terminal in the `pacs-lab` folder:

| System | How |
| :--- | :--- |
| Windows 11 | In File Explorer, right-click the folder and choose **Open in Terminal** |
| macOS | Open Terminal, type `cd` and a space, drag the folder into the window, and press Enter |
| Linux | In the file manager, right-click the folder and choose **Open in Terminal** |

## Run the lab

The commands are the same in PowerShell, Command Prompt, macOS Terminal, and Linux shells. Docker Desktop must be running first.

1. Start the PACS:

   ```
   docker compose up -d
   ```

   The first start downloads the Orthanc image and takes a minute or two.

2. Open <http://localhost:8042> and sign in. The user name is `orthanc` and the password is `orthanc`.

3. Download the test study:

   ```
   docker compose run --rm download
   ```

   It ends with a list of folders: one collection, one patient, two studies, and one series in each study.

4. Send the study to the PACS:

   ```
   docker compose run --rm upload
   ```

   When it prints `Upload finished (HTTP 200)`, refresh the browser. The study list shows two studies for patient `PAN_03`.

5. Open a study and click a viewer button. **View in OHIF** shows the slices. **View in OHIF Volume Rendering mode** adds coronal, sagittal, and 3D views.

The downloaded files stay inside Docker, in a volume called `study`, so they never touch your own folders. That avoids file path limits and file permission problems that differ between systems.

## Stop and reset

| Command | Result |
| :--- | :--- |
| `docker compose down` | Stops the PACS. Your images stay. |
| `docker compose down -v` | Stops the PACS and deletes the images stored in it. The downloaded study stays, so you can run the upload again. |
| `docker compose --profile tools down -v` | Stops the PACS and deletes everything: the stored images and the downloaded study. |

## What runs

| Part | Detail |
| :--- | :--- |
| `pacs` | `orthancteam/orthanc:26.9.1` with Orthanc Explorer 2, OHIF, Stone Web Viewer, and DICOMweb |
| Web interface and REST API | `localhost:8042` |
| DICOM port | `localhost:4242`, AE title `ORTHANC` |
| `download` helper | Runs the NCI Imaging Data Commons tool `idc-index` 0.12.5 in a temporary container and saves the study, plus a ZIP copy, in the `study` volume |
| `upload` helper | Waits until the PACS is ready, then sends the ZIP to the Orthanc REST API with `curl`. Orthanc stores each DICOM file in it. |

Both ports are bound to `127.0.0.1`, so only this computer can reach them. The `orthanc` password is acceptable only for that reason. Do not expose this lab to a network.

## Test data

The lab uses patient `PAN_03` from the CTpred-Sunitinib-panNET collection: two contrast-enhanced abdominal CT studies with 422 images in total, from a GE Discovery CT750 HD scanner with 1.25 mm slices.

**Citation:**
Chen, L., Wang, W., Jin, K., Yuan, B., Tan, H., Sun, J., Guo, Y., Luo, Y., Feng, S.-ting, Yu, X., Chen, M.-hu, & Chen, J. (2022). *Prediction of Sunitinib Efficacy using Computed Tomography in Patients with Pancreatic Neuroendocrine Tumors (CTpred-Sunitinib-panNET)* (Version 1) [Data set]. The Cancer Imaging Archive. <https://doi.org/10.7937/SPGK-0P94>

- **License:** [Creative Commons Attribution 4.0 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). The lab downloads the files unchanged.
- **Data use:** follow the [TCIA Data Usage Policy](https://www.cancerimagingarchive.net/data-usage-policies-and-restrictions/). Credit the dataset whenever you show it, and do not try to identify or contact the people in it.
- **Download service:** NCI Imaging Data Commons. Fedorov, A., et al. (2023). National Cancer Institute Imaging Data Commons: Toward Transparency, Reproducibility, and Scalability in Imaging Artificial Intelligence. *RadioGraphics*, 43(12). <https://doi.org/10.1148/rg.230180>
- **De-identification:** the files carry `Patient Identity Removed = YES` and list the DICOM PS3.15 Annex E options used. The dates were shifted on purpose; the intervals between them were kept.

## Troubleshooting

| Problem | Fix |
| :--- | :--- |
| `docker` is not recognized, or `command not found` | Install Docker, then open a new terminal. |
| `Cannot connect to the Docker daemon`, or an error that mentions `dockerDesktopLinuxEngine` | Start Docker Desktop and wait until it shows that the engine is running. On Linux, start the Docker service. |
| `permission denied` on Linux | Put `sudo` in front of the command, or add your user to the `docker` group. |
| Docker Desktop on Windows asks you to update WSL | Open a terminal as administrator, run `wsl --update`, then restart Docker Desktop. |
| `port is already allocated` | Another program uses port 8042 or 4242. Stop it, or change the first number in the `ports` line, for example `"127.0.0.1:18042:8042"`, and open that port instead. |
| The sign-in prompt keeps returning | Use user `orthanc` and password `orthanc`. |
| `error encountered when reading a file` from the upload | Run the download first (step 3). |
| The download stops with a network error | Run it again. |

## Independence

Polyseminar is an independent education publisher. It is not affiliated with or endorsed by a certification body, professional society, healthcare provider, or imaging vendor. Orthanc, OHIF, TCIA, and IDC are the work of their own authors.
