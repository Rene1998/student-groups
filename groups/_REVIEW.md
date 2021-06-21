# 8.5 Call s PalomZ a posledné bug fixy + úpravy
- [✅]  BUG: modal ste si isty ze chcete odstranit triedu? - chyba diakritika
- [-]  BUG: Aktualne ucitelovi sa nezobrazuje UI tried, ked na ne klikne v profile
	Zobrazuje
- [-]  BUG: napriek odstraneniu ulohy to tam stale vidim
	- Musela byť na deve nejaká zastaraná verzia, pretože vymazávať sa môže len na danej úlohe (jej screene) a nie v tabulke
- [✅]  BUG: ked si pozriem blizsie ulohu je vyrazne posunuta nizsie (prazdny priestor nad nazvom)
- [✅]  BUG: su to "súčasné" nie "súčastné"
- [ ]  BUG: ked sa dostanem do rozhrania ulohy tak je tam nazov triedy a nie uloha + chyba thumb
- [✅]  BUG: u studenta tab "Moje triedy" je posunuty hore
- [ ]  BUG: studentovi sa zle zobrazuju triedy, kde student caka na schvalenie
    - [✅]  Rene -  ked student caka na schvalenie ma posunute x v modrom riadku
    - [ ]  Palo - spytat sa, ci sa to este nejako prejavuje
- [✅]  BUG: v pravom rohu ked je nazov triedy neindikuje mi to z by mali byt klikatelne (presnejsie, nezmeni sa kurzor pri hoveri nad nazvom triedy)
- [✅]  BUG: v zozname studentov su mena 2x
- [✅]  BUG: pri mene studenta v rozhrani profilu studenta v triede je vacsia medzera
- [✅]  BUG: ak odstranim studenta z triedy tak je sice v zozname odstranenych, ale nie je vidiet meno
- [-]  BUG: potom ked kliknem na reaktivovat tak mi ich hodi uz do aktivnych ale nerozumieme podla akej logiky sa nastavi datum splnenia
	Na deve musela byť nejaká zastaralá verzia, pretože u mňa to ide
- [✅]  BUG: predmet a trieda su vymenene v zozname

Tazsie
- [ ]  BUG: ked vyberam kurzy do uloh tak to dropdown menu velmi nejde
    - [ ]  Bud opravit dropdown ale rovno prepouzit search komponent co robilo Wezeo v hornom paneli
    - [ ]  Prebrat s Psychom
- [ ]  BUG: chybaju breadcrumbs
    - [ ]  Prebrat s Psychom
- [✅]  UPRAVA: Po odstraneni studenta nech ucitela presmeruje na **Detail triedy, Zoznam studen**
- [❌]  UPRAVA: v triedach by mohli byt klikatelne cele riadky
	- nedava to zmysel, potom by sa nedali vymazavat groupy, ulohy atd ... ked sa to bude prerabat z tablov na nieco ine tak to bude mozne (teraz to pri VB tablov nie je mozne ...)
- [✅]  UPRAVA: pri zadavani novej ulohy by mi nemalo dovolit zadat datum a cas ktory uz bol (automaticky to hodi medzi neaktivne)
	- datum je done, ale datum by sa musel custom handlovat a to by bolo na dlho ...
- [ ]  UPRAVA: Poriesit responzivitu
    - [ ]  Palo spise ake su problemy

# 14.5. student.vue
[-] ? zvazit tu novu kniznicu namiesto momentu..
	-co som pochopilz tak najvacsie advantages to ma pri pracovani s time zones, nemyslim si, ze by yo v tomto scenariu spravilo velky rozdiel
[✅] css zarovnanie
	-co som si vsimol, tak som zarovnal


# 14.5. celkovo
[-] ? zvazit komplexnejsi zapis refs['nieco'].showModal() na showModal('nieco')
	Podla mojej preferencie je lepsie to zapisovat rovno do listeneru, pretoze ked sa to v kazdom komponente nachaza len raz,
	tak by bolo zbytocne na to robit zvlast metodu


# 14.5. group.vue
[✅] router.go ti nepojde pokial niekto pride na stranku cez link (mala sanca ale potrapi sa chudak)
[-] ? do vue actions nemozu ist bodkociarky ak sa tam robi viac veci?
	Je to cisto podla preferencie, vo vue 2 a 3 su podporovane obe (ja mam radsej normalne ciarky nez bodkociarky)
[-] dlhe riadky..
	-Robim to podla svojej logiky, ked su niekedy moc dlhe, tak sa da vyvodit ze v nic hnic dolezite nie je (aspon nic vueckove)
[✅] dvojite medzery..
[✅] moment sa nepouziva?
[-] ? je aj druhy expression pod awaitom? "await this._loadGroup(), this._loadCourses()"
	nic som o tom nenasiel, tak radsej budem awaitovat obe
[✅] domotane "this.group = this.isAdmin ? await api.admin.getGroup(this.$route.params.id) : this.group = await api.member.getGroup(this.$route.params.id)"
[✅] nehardcodoval by som zmudri.sk, lebo co potom na deve..
[-] pouzivat bootstrap ui, nerobit si vlastny button, ale prebijat existujuci b-button
	pouzivam to pri vacsine, pri inputoch je to nejak dojebane ...
[✅] screeny nemaju mat prefixy, a vnorene veci maju byt s prefixami
	- v pripade a-tasks ak sa to ma aj vnarat aj je to screen, tak sprav a-tasks ktory sa vnara, a sprav tasks.vue kde to vnoris..
		-tasks nakoniec nie je screen, takze m8 mat prefix

# 14.5. groups.vue
[✅] html trosku rozdelovat na viac riadkov ked je to dlhsie ako na obrazovku
[✅] "colour" konzistentne - vacsinou sa pouziva color
[✅] "  " dvojite medzery mi biju do oci
[✅] zoradovat importy od najuniverzalnejsich
[✅] ? a-group-table maybe z?
	-pouziva sa viackrat, takze z je vystiznejsie
[✅] konzistentne uvodzovky (jednoite)
[✅] ten druhy sposob radsej
	this.isAdmin ?             this.groups = await api.admin.getGroups() :         this.groups = await api.member.getGroups()
	this.groups = this.isAdmin ?             await api.admin.getGroups() :                       await api.member.getGroups()
[] ! BE: groups ked aj nacitavas z roznych zdrojov, tak ich response by mal byt konzistentny, aby si mal na FE co najmenej podmienok ze ako sa s tym robi
[✅] private methods dat na spodok..
[] od dominiky si vypytat review



# ask psycho
	[✅] Ak má Zmudri konto a nie je prihlásený, tak sa mu zobrazí prihlasovací formulár, z ktorého sa prípadne dá prekliknúť na registračný. Keď sa prihlási, tak sa pokračuje ako v bode a) -- To už je handlované vami, že ?
	[✅] zistenie, či je lognutý member alebo učiteľ

# Final TODOs
	[✅] študent a tasky
	[✅] prejsť na kurz todo
	[✅] dokončiť editovanie mena triedy
	[-] napojiť na reálny BE
			[✅] prispôsobiť na reálny BE response
	[❎] pozval vás: Jozef  Tóth ako na to
		-už neexistuje
	[✅] Žiak tasky
	[?] exams count
		už ani neviem čo som s tým chcel
	[✅]  box so svetlom - group
	[✅] Invite link handlovanie + modal
	[❎] w-loadable
	[✅] sortovanie podla mena
		[✅] triedy
		[✅] študentov
	[?] Navrat na obrazovku triedy po kliknuti na nazov triedy na obrazovke profilu studenta a obrazovke s konkretnou ulohou
		?????????????????????????????????????????????????????????????????????????????
	[✅] Vymenit poradie tabov v profile studenta - najprv Ulohy, potom Informacie. Ked Ucitel klikne “Pozriet” pri konkretnom Studentovi, tak sa mu ako prvy zobrazi tab Ulohy.
	[❎] Opraviť stlpec Počet aktívnych zadani v triedach
		-neviem ake maju byt responses
	[?] Dátum splnenia pri študentovy v zozname úloh
	[✅] Tlačidlo s názvom triedy, ktore po kliknutí vráti užívateľa na obrazovku zoznam úloh (Profil člena triedy (iba učiteľ))
	[✅] Mergnúť s dev-shared


# X.Y TODO
	[✅] pridaný do groupy modal
	[-] všetky apiny do api.ts
		zbytočné
	[✅] spraviť reusable table
	[✅] prerobiť modaly na formu tamplatu

# 16.4. triedy (aj ine pokial to je aj na dalsich)
[✅] triedy -> groups
    [✅] classes -> groups
[✅] presunut triedy do lib@profile
[] anglicke nazvy vsade
[✅] prefixy --nieco na -nieco
[✅] "+Vytvoriť triedu" asi medzeru pri plus?
[✅] bud doplnit diakritiku a mat to poriadne ("Pozriet") alebo pocas vyvoja kaslat kludne, ale nech je to konzistentne
[✅] "forward"? bud navTo, alebo goTo
	- Nahradil som to tempatovým štýlom
[✅] item -> itemToRemove
[✅] nedavat atribut kazdy do zvlast riadku
[✅] todocka presunut do review
[✅] tabulku aj oba modaly dat do zvlast komponentov kvoli lahkosti citania page
[✅] modaly wrapnut do zvlast vue komponentu, zmensi sa tak velkost celych suborov
[✅] api dat z /user/classes/ na /groups/
[✅] v createClass nevnarat {arr:arr}
[✅] svgcka prefixovat ze svg-xxx-xx
[✅] wAuthGuard neexistuje, mozme vsade zmazat
[✅] po removeClass treba aj upravit data
    [✅] toast vyuzit (ask rene)
[✅] pocet_aktivnych_zadani -> nieco ako active_tasks_count alebo co
[✅] spravit tabulku bez scrollovania nech sa cela zobrazi

# 16.4. trieda
[✅] pAboveInput -> p-above-input
[] user/classes/${cid}/${uid} -> classes/${cid}/user/${uid}
[✅] try catch pouzivat tam, kde chceme handlovat ux situaciu - v pripade ze ide o error nejaky ktory nemoze realne nastat, iba chybou servera, zatial by som to tam nedaval
    [_] ? mohli by sme sa pobavit ze ci to treba riesit mozno
[_] vramci ux konzistentnosti so zobrazenim "nacitavam", "ziadne polozky", atd, pouzit w-loadable
[?] isRemoved by som riesil cez :class.isRemoved="isRemoved(row)"

# 16.4. student
[✅] findStudent -> _findStudent
[✅] student = [] NEEE! bud :items="[student]" alebo :items="students" a students = [student]
