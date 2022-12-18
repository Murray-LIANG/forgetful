# WSL


## Issues

### 1. wslregisterdistribution failed with error 0x80080005.

1. Run PowerShell as admin.
2. `.\sc.exe stop LxssManager` then `.\sc.exe start LxssManager`.
3. You may find `LxssManager` is under `STOP_PENDING` status after you stop it.
4. `.\sc.exe queryex LxssManager`
```powershell
PS C:\WINDOWS\system32> .\sc.exe queryex LxssManager

SERVICE_NAME: LxssManager
        TYPE               : 30  WIN32
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
        PID                : 27192
        FLAGS              :
```
5. Open `Task Manger` and switch to tab `Details`. Find the `svchost.exe` with above PID 27192. Then `End process tree` in the right click menu.
6. You can `.\sc.exe start LxssManager` now.

### 2. wsl the user has not been granted the requested.
1. Run PowerShell as admin.
2. `Get-Service vmcompute | Restart-Service`
