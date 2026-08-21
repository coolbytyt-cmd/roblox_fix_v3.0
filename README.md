![Windows](https://img.shields.io/badge/Windows-10%20%7C%2011-blue?logo=windows)
![License](https://img.shields.io/badge/License-MIT-green)
![Version](https://img.shields.io/badge/version-1.0-orange)

by: lzeo
              ROBLOX CDN FIXER
         Автоматический фикс для Roblox
                   

📌 ДЛЯ ЧЕГО ЭТОТ ФАЙЛ?

Когда Roblox не грузится, виснет на загрузке, не показывает
текстуры и аватары — проблема часто в том, что компьютер
не может найти правильный IP-адрес серверов Roblox.

Этот скрипт сам находит рабочие IP и прописывает их в
системный файл hosts. После этого игра начинает грузиться.

============================================================

⚙️ ЧТО ОН ДЕЛАЕТ?

1. Делает бэкап файла hosts (на всякий случай)
2. Удаляет старые записи Roblox
3. Проверяет доступность IP-адресов через ping
4. Прописывает в hosts только работающие IP
5. Если ничего не нашёл — опрашивает публичные DNS
6. Сбрасывает DNS-кэш и Winsock

============================================================

🚀 КАК ЗАПУСТИТЬ?

1. Скачай файл roblox_fix.bat
2. Нажми на него ПРАВОЙ кнопкой мыши
3. Выбери «Запуск от имени администратора»
4. Дождись надписи [DONE]
5. ПЕРЕЗАГРУЗИ КОМПЬЮТЕР
6. Запускай Roblox

============================================================

⚠️ ВАЖНО!

- Запускать ТОЛЬКО от имени администратора
- После запуска ОБЯЗАТЕЛЬНО перезагрузить ПК
- Если не помогло — запусти ещё раз через час
- Если антивирус ругается — добавь файл в исключения

============================================================

🔄 КАК ОТКАТИТЬ?

Если что-то пошло не так, открой командную строку
от администратора и вставь эту команду:

copy "C:\Windows\System32\drivers\etc\hosts.backup" "C:\Windows\System32\drivers\etc\hosts"

============================================================

❓ НЕ РАБОТАЕТ?

Попробуй:
1. Отключить антивирус на 5 минут
2. Запустить скрипт ещё раз
3. Использовать в связке с Zapret

============================================================

📱 Вопросы telegram: @lzeo3


============================================================

сомнения? вот исходный код!

! исходный код для bat:

============================================================


@echo off
chcp 1251 >nul
title Roblox CDN Fixer
setlocal enabledelayedexpansion

echo ============================================================
echo      ROBLOX CDN FIXER - Auto IP selector
echo ============================================================
echo.

net session >nul 2>&1
if %errorlevel% neq 0 (
    echo [ERROR] Run as Administrator!
    echo Right click -> Run as administrator
    pause
    exit /b 1
)

if not exist "%windir%\System32\drivers\etc\hosts.backup" (
    copy "%windir%\System32\drivers\etc\hosts" "%windir%\System32\drivers\etc\hosts.backup"
    echo [BACKUP] hosts.backup created
) else (
    echo [BACKUP] already exists
)
echo.

findstr /v /i "roblox.com rbxcdn.com" "%windir%\System32\drivers\etc\hosts" > "%temp%\hosts_clean.tmp"
copy /y "%temp%\hosts_clean.tmp" "%windir%\System32\drivers\etc\hosts" >nul
del "%temp%\hosts_clean.tmp"
echo [CLEAN] Old Roblox records removed
echo.

set DOMAINS=tr.rbxcdn.com roblox.com rbxcdn.com

set IP_LIST=18.64.211.78 18.64.211.88 18.64.211.103 18.64.211.77 18.64.211.89 18.64.211.90 18.64.211.91 18.64.211.92 18.64.211.93 18.64.211.94 18.64.211.95 18.64.211.96 18.64.211.97 18.64.211.98 18.64.211.99 18.64.211.100 23.207.188.118 23.207.188.119 23.207.188.120 23.207.188.121

echo ============================================================
echo [SCANNING] Checking available IPs...
echo ============================================================
echo.

set FOUND_IPS=
set COUNT=0

for %%i in (%IP_LIST%) do (
    ping -n 1 -w 500 %%i >nul 2>&1
    if !errorlevel! equ 0 (
        echo [OK] %%i - available
        set FOUND_IPS=!FOUND_IPS! %%i
        set /a COUNT+=1
    ) else (
        echo [NO] %%i - unavailable
    )
)

echo.
echo [RESULT] Found %COUNT% working IPs
echo.

if %COUNT% equ 0 (
    echo [ERROR] No working IPs found!
    echo Trying alternative method via nslookup...
    echo.
    goto :alternative
)

echo [WRITE] Adding found IPs to hosts...
echo. >> "%windir%\System32\drivers\etc\hosts"
echo # === ROBLOX CDN FIX (auto %date% %time%) === >> "%windir%\System32\drivers\etc\hosts"

for %%a in (%FOUND_IPS%) do (
    echo %%a tr.rbxcdn.com >> "%windir%\System32\drivers\etc\hosts"
    echo %%a roblox.com >> "%windir%\System32\drivers\etc\hosts"
    echo %%a rbxcdn.com >> "%windir%\System32\drivers\etc\hosts"
)

echo # === END OF BLOCK === >> "%windir%\System32\drivers\etc\hosts"
echo [WRITE] Done
goto :finish

:alternative
echo [ALTERNATIVE] Using public DNS for resolution...
echo.

set DNS_SERVERS=8.8.8.8 1.1.1.1 208.67.222.222 77.88.8.8
set ALT_IPS=

for %%d in (%DNS_SERVERS%) do (
    for %%h in (tr.rbxcdn.com) do (
        nslookup %%h %%d 2>nul | findstr "Address:" | findstr /v "%%d" > "%temp%\ip_temp.txt"
        for /f "tokens=2 delims=:" %%i in (%temp%\ip_temp.txt) do (
            set "IP=%%i"
            set "IP=!IP: =!"
            if not "!IP!"=="" (
                ping -n 1 -w 300 !IP! >nul 2>&1
                if !errorlevel! equ 0 (
                    echo [OK] !IP! - available via %%d
                    set ALT_IPS=!ALT_IPS! !IP!
                )
            )
        )
    )
)

if "!ALT_IPS!"=="" (
    echo [ERROR] No working IPs found via alternative DNS.
    echo Manual steps:
    echo 1. Open https://whatismyipaddress.com/hostname-ip/tr.rbxcdn.com
    echo 2. Find working IP and add manually to hosts
    echo 3. Run ipconfig /flushdns
    pause
    exit /b 1
)

echo [WRITE] Adding IPs from alternative source...
echo. >> "%windir%\System32\drivers\etc\hosts"
echo # === ROBLOX CDN FIX (alt %date% %time%) === >> "%windir%\System32\drivers\etc\hosts"

for %%a in (!ALT_IPS!) do (
    echo %%a tr.rbxcdn.com >> "%windir%\System32\drivers\etc\hosts"
    echo %%a roblox.com >> "%windir%\System32\drivers\etc\hosts"
    echo %%a rbxcdn.com >> "%windir%\System32\drivers\etc\hosts"
)

echo # === END OF BLOCK === >> "%windir%\System32\drivers\etc\hosts"

:finish
echo.
echo ============================================================
echo [DONE] IPs added to hosts
echo.
echo [ACTION] Flushing DNS cache...
ipconfig /flushdns
echo [ACTION] Resetting Winsock...
netsh winsock reset
echo.
echo ============================================================
echo [RECOMMENDATIONS]
echo 1. Close all browsers and Roblox.
echo 2. Restart PC for full apply.
echo 3. If not working - run script again after 1 hour.
echo 4. Backup location: %windir%\System32\drivers\etc\hosts.backup
echo.
echo [RESTORE] To rollback changes:
echo copy "%windir%\System32\drivers\etc\hosts.backup" "%windir%\System32\drivers\etc\hosts"
echo.
pause
