# Boolean Mantığı Operatörleri

**AND (& veya AND)**: Seri bağlantı (Tüm koşullar sağlanmalı).

**OR (OR)**: Paralel bağlantı (Koşullardan biri sağlansa yeter).

**NOT (NOT veya !)**: Kapalı kontak / Tersleme (Sinyalin zıttı).

**XOR (XOR)**: Özel VEYA (Sadece biri TRUE ise çıkış verir, ikisi de TRUE veya FALSE ise çıkış vermez).


# Mühürleme (Latching) Yöntemleri

**#1 Boolean Denklemi ile:** (Makbul olan)
// Çıkış := (Başlatma VEYA Kendisi) VE (Durdurma Değil)
Motor := (Start OR Motor) AND NOT Stop;

**#2 IF / THEN Koşulları ile:**
// Set işlemi (Başlatma)
IF Start THEN
    Motor := TRUE;
END_IF;

// Reset işlemi (Durdurma - Öncelikli)
IF Stop THEN
    Motor := FALSE;
END_IF;


# Good-to-know:

1- AND işlem önceliği OR'dan yüksek.

2- Atamalar tek satırda (Atlanmaması için).

3- Cycle, yukarıdan aşağıya çalışıyor, en alttaki satır öncelikli.

4- NC ve NO buton kullanımına dikkat (Reelde stop için NC buton kullanımı yaygın).


# The Code:

#iq_bRun := (#i_bStart OR #iq_bRun) AND NOT #i_bStop;