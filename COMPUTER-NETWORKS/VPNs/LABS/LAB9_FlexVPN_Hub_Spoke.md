Genralnie, aby łatwiej było zapamiętać architekturę FlexVPN i wyjść od ogółu do szczegółu, 
tworzymy sobie strukturę drzewa. Główny pień to interfejsy. Do nich dowiązujemy Profil Fazy 2 (IPsec). 
Profil Fazy 2 ma w sobie zagnieżdżony Profil Fazy 1 (IKEv2). 

Wszystko jest spięte jak po sznurku. Router, zestawiając tunel, leci dokładnie po tej ścieżce. 
Od głównych profili odchodzą "odnogi/gałęzie" (hasła, autoryzacja, kryptografia), które trzeba 
wcześniej przygotować. 

UWAGA: Mapę CZYTAMY z góry na dół (aby zrozumieć logikę), ale KONFIGURUJEMY z dołu do góry 
(zgodnie z numeracją KROK 1 -> KROK 9), przygotowując najpierw "cegiełki", a na końcu spinając je w całość.

==========================================================================================
                      MAPA MYŚLI: FlexVPN HUB-AND-SPOKE (CENTRALA)
==========================================================================================

[INTERFEJSY] (KROK 9 - Finałowe spięcie)
 ├── interface GigabitEthernet1 (Underlay - wyjście na świat)
 ├── interface Loopback1 (LAN - adres pożyczany do tuneli)
 └── interface Virtual-Template 1 type tunnel (Maszyna klonująca dla oddziałów)
      │
      └── tunnel protection ipsec profile FLEXVPN_PROFILE
           │
           ▼
[PROFIL FAZY 2 - IPsec] (KROK 8 - Szyfrowanie danych)
crypto ipsec profile FLEXVPN_PROFILE
 ├── set transform-set FLEXVPN_TRANSFORM  <--- (KROK 7: crypto ipsec transform-set ...)
 │
 └── set ikev2-profile FLEXVPN_PROFILE
      │
      ▼
[PROFIL FAZY 1 - IKEv2] (KROK 6 - Mózg operacji i autoryzacja)
crypto ikev2 profile FLEXVPN_PROFILE
 ├── match identity remote address 0.0.0.0 (Wpuszczaj każdego)
 ├── authentication remote pre-share (Żądaj hasła od Spoke'a)
 ├── authentication local pre-share  (Przedstaw się swoim hasłem)
 ├── virtual-template 1              (Wskaż maszynę klonującą)
 │
 ├── ODNOGA 1: BAZA HASEŁ
 │    └── keyring local FLEXVPN_KEYRING
 │         │
 │         ▼
 │        [KEYRING] (KROK 3 - Pęk kluczy)
 │        crypto ikev2 keyring FLEXVPN_KEYRING
 │         └── peer FLEVPNPeers (address 0.0.0.0 0.0.0.0)
 │              ├── pre-shared-key local cisco123
 │              └── pre-shared-key remote cisco123
 │
 └── ODNOGA 2: WYPRAWKA DLA ODDZIAŁU (IP + Trasy)
      └── aaa authorization group psk list FlexAuth HUBPolicy
           │
           ▼
          [AAA & AUTHORIZATION POLICY] (KROK 5 - Pakowanie plecaka)
          crypto ikev2 authorization policy HUBPolicy
           ├── route set interface
           ├── pool FlexPool                  <--- (KROK 4B: ip local pool FlexPool 10.1.1.2...)
           └── route set access-list FlexTraffic <--- (KROK 4C: ip access-list standard FlexTraffic...)
           │
           *Wymaga włączenia AAA:             <--- (KROK 4A: aaa new-model / aaa authorization network FlexAuth local)

==========================================================================================
* UWAGA DO KRYPTOGRAFII (KROK 1 i 2):
Polityka IKEv2 (crypto ikev2 policy) oraz Proposal (crypto ikev2 proposal) "wiszą" 
w konfiguracji globalnej. Router sam po nie sięga na samym początku negocjacji. 
Nie przypinamy ich bezpośrednio do żadnego profilu!
==========================================================================================