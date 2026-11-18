# The Haven's Home Authentication System

A local facial-recognition access-control system built with Python, OpenCV and Flask. It recognises enrolled household members, denies unknown visitors, keeps an access history, and (when armed) raises an alarm and emails the homeowner a photo of the unrecognised person.

**Group 2 · CYB 301**

> This project is a redesign and extension of the open-source *Home Security System* by [ayomidedavid](https://github.com/ayomidedavid/Home-Owner-Authentication-System). The original authors' facial-recognition engine, alarm logic and project structure are the foundation of this work. The original repository did not include a licence file when we cloned it.

---

## What we changed

| Area | Change |
|---|---|
| Dashboard | New light-blue, responsive layout that works on phones and tablets; team section; live **ACCESS GRANTED / ACCESS DENIED / VERIFYING** banner. |
| Decisions | Per-frame results are smoothed over several frames, so a person who is still turning to the camera is shown as *Verifying* and not wrongly denied. |
| Matching | Photos are grouped by person; a face is accepted only if the distance is under a threshold **and** clearly better than the next closest person. |
| Access history | New log of every granted and denied attempt, with a PIN-protected clear button. |
| Fixes | Deleting an enrolled person now works; Windows webcam backends are tried before falling back to the simulator; a missing MySQL server no longer freezes the live feed. |
| Security | Flask secret key can be set through the `SECRET_KEY` environment variable; incident and access logs can only be cleared with the PIN. |

## How recognition works

1. **Detection:** OpenCV Haar cascades locate faces in each camera frame.
2. **Preparation:** each face is cropped, resized to 200 × 200 and histogram-equalised.
3. **Recognition:** an LBPH (Local Binary Patterns Histograms) model compares the face with every enrolled sample and returns a **distance score**. Lower is better; it is *not* a percentage.
4. **Decision:** a person is **granted** when the best distance is at most `match_threshold` (default 60) and beats the next closest person by at least `match_margin` (default 8). Otherwise the person is **denied**.
5. **Confirmation:** the decision must hold across several frames before it is shown, logged or used to raise an alarm.

In our group's tests, enrolled members were granted with distance scores of roughly 46–50, and unenrolled people were denied with scores of 71 and above. This was a small test and lighting-dependent.

## Requirements

- Windows 10/11 (also runs on other systems that support OpenCV)
- Python 3.11 or newer
- A webcam
- Git

## Setup

```bash
git clone <your-repository-url>
cd <repository-folder>
python -m venv .venv
.\.venv\Scripts\Activate.ps1        # Windows PowerShell
pip install -r requirements.txt
python run.py
```

Open **http://127.0.0.1:5000** in your browser.

## Using the system

1. **Enrol people:** *Household → Add Person*. Take a snapshot or upload a photo, type the full name, choose the role (Homeowner, Parent, Child, Family Member) and click *Enroll Face*. Use the **exact same name** for every photo of the same person, and enrol 2–3 photos (straight, slightly left, slightly right) in the lighting you will use.
2. **Arm the system:** click **Home** or **Away**. When armed, an unrecognised person triggers the siren, a photo snapshot, an email alert and an entry under *Security Incidents*. Click **Off** and enter the PIN to disarm.
3. **Watch decisions:** the banner above the camera shows *Access granted*, *Access denied* or *Verifying*. The *Activity Log → Access History* tab records every decision.
4. **Settings:** set the owner name and email, optional SMTP, and the PIN in the *Settings & DB* tab.

> The **default PIN is `1234`. Change it in Settings before use.** Your PIN, email and SMTP password are stored in `data/config.json`, which is excluded from Git.

## Optional email alerts (SMTP)

Without SMTP, alerts are saved in the *Dispatched Notifications Outbox*. To send real email with Gmail, create an App Password (Google Account → Security → 2-Step Verification), then in Settings tick *Enable Live SMTP*, use host `smtp.gmail.com`, port `587`, your Gmail address as username and the App Password as password. Use *Send Test Email* to check.

## Optional tuning (`data/config.json`)

| Key | Default | Meaning |
|---|---|---|
| `match_threshold` | `60` | Highest distance still accepted. Lower is stricter. |
| `match_margin` | `8` | Required gap to the next closest person. |
| `show_match_distance` | `true` | Show the distance on the video label. Set `false` for presentations. |
| `unknown_confirm_seconds` | `2` | How long a face must stay unrecognised before it is denied. |
| `confirm_votes` | `3` | Minimum matching frames before access is granted. |
| `access_log_cooldown` | `15` | Seconds between log entries for the same person. |
| `alert_cooldown_seconds` | `20` | Seconds between repeated intruder alerts. |

## Project structure

```
core/      camera, face engine, database, security controller, mailer
web/       Flask app (app.py) and dashboard (templates/index.html)
data/      enrolled faces, snapshots, database and settings (not committed)
run.py     starts the application
test_system.py   automated tests
```

## Database

The application uses SQLite automatically. MySQL/MariaDB is optional; if it is not running the system falls back to SQLite.

## Privacy and limitations

- Face images are biometric data. They are stored only on the local machine, and `data/` is excluded from Git. Obtain consent before enrolling anyone.
- LBPH is sensitive to lighting, distance and head angle. Enrol under the lighting you will use.
- The system has **no liveness detection**; a printed photo or a phone screen may be accepted.
- The door-unlock message is a simulation. No physical lock is connected.
- The system was tested with a small group in one environment and should not be relied on for real-world security.

## Team

- Onaeko Anjolaoluwa
- Aboaba Mobolaji
- Adeniba Oluwaseyintanfunmi
- Yakubu Jesse
- Ijitola Oluwanifemi
- Felix Joseph
- Adegbehingbe Ronald
- Yomi-Aworonke Tioluwanimi


## Credits

Original project: *Home Security System* by ayomidedavid. Redesign, access logging, matching improvements and dashboard by Group 2, CYB 301.