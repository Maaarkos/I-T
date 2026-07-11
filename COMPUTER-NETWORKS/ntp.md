NTP operates on 123 UDP port

it is based on the hierarchical system like below:

<div align="center">
  <a href="IMAGES/ntp.png" target="_blank">
    <img src="IMAGES/ntp.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

Keep in mind that even if we save the time to NVRAM using the clock set command, the clock will reset after a reboot. This is because the system reads the time from the motherboard battery. Sometimes, the device doesn't even have one. However, there are commands to synchronize this battery with the system time

Najlepiej ustawic serwer NTP

Ponizej przydatne komendy:

<div align="center">
  <a href="IMAGES/ntp-architecture.png" target="_blank">
    <img src="IMAGES/ntp-architecture.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

<div align="center">
  <a href="IMAGES/ntp-1.png" target="_blank">
    <img src="IMAGES/ntp-1.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

<div align="center">
  <a href="IMAGES/ntp-2.png" target="_blank">
    <img src="IMAGES/ntp-2.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

<div align="center">
  <a href="IMAGES/ntp-3.png" target="_blank">
    <img src="IMAGES/ntp-3.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>