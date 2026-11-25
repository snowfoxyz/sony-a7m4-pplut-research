# A7M4 camera read-only ulogio access (2026-11-25)

Purpose: attempt official read-only camera observation of the Picture Profile
menu / PPLUT provider, without flashing or writing properties.

## Environment

- Camera: ILCE-7M4
- Firmware version string read from `/version.txt`:
  `30181_18.2026041801`
- Windows, Sony-PMCA-RE (local `Sony-PMCA-RE`), venv.
- Camera USB mode: Mass Storage; PMCA switches to service mode.
- Read-only Jiritsu commands only: SET_FILE_PATH / GET_FILE2.
- No `bk w`, no `bk s`, no fastboot write.

## Successful reads

```text
/version.txt
19 bytes
sha256 b5377b8e08ba1f7a1105bcb6af07187414a5493a3d4622f02b7ad8a2578c87e5
```

```text
/setting/ulogio.bin
2764 bytes
sha256 452e24f8394e7d932178fbf7de8cf547954861f44ae25400a82867892827cee4
```

The `/setting/ulogio.bin` hash exactly matches the local
`seed-setting/ulogio.bin` (same SHA-256). It is a **ulogio configuration**,
not a runtime log. It names:

```text
/setting/filter.bin
/log/escapelog.bin
/log/ulogio_blog.dat
/log/ulogio_dlog.dat
/log/ulogio_klog.txt
/log/ulogio_tlog.txt
```

## Failed reads

All of the following returned:

```text
GET_FILE2 status=0x10000006
```

```text
/log/ulogio_blog.dat
/log/ulogio_dlog.dat
/log/ulogio_klog.txt
/log/ulogio_tlog.txt
/log/escapelog.bin
/setting/filter.bin
```

Additional observations:

```text
/proc/mounts        returns 0 bytes
/proc/self/mountinfo returns 0 bytes
/etc/fstab          GET_FILE2 0x10000006
```

## Interpretation

Verified:

```text
official service-mode file read works
/setting/ulogio.bin is only the ulogio configuration
runtime /log files are not exposed/present in this service-mode session
```

Not established:

```text
whether /log files exist only while the normal application/UI is running
whether ulogio runtime capture is disabled on retail firmware
whether service mode stops the application before logs are created
```

Because the log files are absent, the camera read-only route does **not**
currently provide the UI candidate/provider events needed for the PPLUT
binding question.

## Boundary

- No camera write, no firmware modification, no flash.
- No PPLUT patch.
- The camera was returned to normal mode after each service session; service
  mode disconnects USB after the script exits.

## Next

Continue only with targeted official/static comparison or a runtime that can
observe the menu. Do not treat the absent logs as evidence that PPLUT exists or
does not exist.

