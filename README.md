# ==========================================
# 1. MERGE CONFLICT (Çakışma Çözümü)
# ==========================================
# Senaryo: İki farklı dalda aynı dosya değiştirilir, çıkan çakışma manuel çözülür.

# 1. Master dalında ilk dosyanın oluşturulması
echo "Ana Arayuz V1" > arayuz.txt
git add arayuz.txt
git commit -m "feat: ilk tasarim eklendi"

# 2. Yeni dalda (feature/login) değişiklik yapılması
git checkout -b feature/login
echo "Login Ekrani Eklendi" > arayuz.txt
git add arayuz.txt
git commit -m "feat: login ekrani eklendi"

# 3. Master dalına dönüp AYNI dosyanın farklı şekilde değiştirilmesi
git checkout master
echo "Ana Arayuz V2 Guncellemesi" > arayuz.txt
git add arayuz.txt
git commit -m "feat: master guncellendi"

# 4. Çakışmayı tetikleme
git merge feature/login

# OLUŞAN ÇAKIŞMA UYARISI (Kanıt):
# warning: Cannot merge binary files: arayuz.txt (HEAD vs. feature/login)
# Auto-merging arayuz.txt
# CONFLICT (content): Merge conflict in arayuz.txt
# Automatic merge failed; fix conflicts and then commit the result.

# 5. Çakışmanın manuel çözülmesi ve birleştirilmesi
# (arayuz.txt dosyası açılarak "Ana Arayuz V2 ve Login Ekrani Birlestirildi" olarak düzenlendi)
git add arayuz.txt
git commit -m "Merge branch 'feature/login' into master (Conflict Resolved)"

# İŞLEM SONRASI OLUŞAN GERÇEK GİT GRAFİĞİ VE LOG (Kanıt):
# *   b8164de (HEAD -> master) Merge branch 'feature/login' into master (Conflict Resolved)
# |\  
# | * e671385 (feature/login) feat: login ekrani eklendi
# * | 5d0cdce feat: master guncellendi
# |/  
# * 28b354c feat: ilk tasarim eklendi

# ==========================================
# 2. REBASE & SQUASH (Geçmişi Temizleme)
# ==========================================
# Senaryo: Gereksiz WIP commit'lerini tek bir anlamlı commit altında toplama.

# 1. Yeni dal oluşturma ve gereksiz commitler atma
git checkout -b feature/data-pipeline
git commit --allow-empty -m "wip: pipeline basladi"
git commit --allow-empty -m "wip: veriler cekildi"
git commit --allow-empty -m "wip: hatalar duzeltildi"

# 2. Son 3 commit'i interaktif olarak tek commit'te birleştirme (Squash)
# (Editörde pick, s, s yapılarak isim "feat: data pipeline altyapisi tamamlandi" olarak değiştirildi)
git rebase -i HEAD~3

# İŞLEM SONRASI OLUŞAN GİT LOG (Kanıt):
# 4ffe5fc (HEAD -> feature/data-pipeline) feat: data pipeline altyapisi tamamlandi
# b8164de (master) Merge branch 'feature/login' into master (Conflict Resolved)

# ==========================================
# 3. GÜVENLİK (Geçmişi Temizleme / filter-repo)
# ==========================================
# Senaryo: Yanlışlıkla eklenen şifre/gizli dosyanın (.env) Git tarihinden kalıcı olarak silinmesi.

# 1. İçinde şifre olan .env dosyasının yanlışlıkla eklenip commitlenmesi
echo "DB_PASSWORD=cokgizlisifre" > .env
git add .env
git commit -m "hata: gizli dosya eklendi"

# 2. Üzerine normal bir commit daha atılması (şifrenin geçmişte kalması için)
echo "Normal bir dosya" > normal.txt
git add normal.txt
git commit -m "feat: normal dosya eklendi"

# 3. git filter-repo aracı ile .env dosyasının tüm tarihten sonsuza dek silinmesi
git filter-repo --path .env --invert-paths --force

# İŞLEM SONRASI OLUŞAN GİT LOG (Kanıt):
# (Dikkat: "hata: gizli dosya eklendi" commiti tarihten tamamen silinmiştir)
# 3b4322e (HEAD -> master) feat: normal dosya eklendi
# b8164de Merge branch 'feature/login' into master (Conflict Resolved)
# 5d0cdce feat: master guncellendi

# ==========================================
# 4. STASH (Yarım Kalan İşi Rafa Kaldırma)
# ==========================================
# Senaryo: Bitmemiş bir çalışmanın rafa kaldırılıp (stash), sonra geri alınıp (pop) tamamlanması.

# 1. Yarım bir dosya oluşturup Git'e ekleme
echo "Yarim kalan veritabani kodu" > veritaban.txt
git add veritaban.txt

# 2. Acil bir durum için yarım işi rafa kaldırma
git stash push -m "yarim-veritabani-isi"

# 3. İşi raftan geri alma (pop) ve tamamlayıp commitleme
git stash pop
echo " - Hatalar giderildi ve kod tamamlandi" >> veritaban.txt
git add veritaban.txt
git commit -m "feat: veritabani baglantisi tamamlandi"

# İŞLEM SONRASI OLUŞAN GİT LOG (Kanıt):
# bce3464 (HEAD -> master) feat: veritabani baglantisi tamamlandi
# 3b4322e feat: normal dosya eklendi

# ==========================================
# 5. CHERRY-PICK (Belirli Bir Commit'i Çekme)
# ==========================================
# Senaryo: Başka bir dalda (hotfix-dali) yapılan kritik bir düzeltmeyi, ana dala (master) cımbızla çekme.

# 1. Yeni bir dalda acil düzeltme yapılması
git checkout -b hotfix-dali
echo "Kritik sistem hatasi cozuldu" > acil_cozum.txt
git add acil_cozum.txt
git commit -m "fix: acil sistem hatasi giderildi"

# 2. Ana dala dönüp o düzeltmeyi cımbızla (cherry-pick) çekme
git checkout master
git cherry-pick hotfix-dali

# İŞLEM SONRASI OLUŞAN GİT LOG (Kanıt):
# * 1537f1d (HEAD -> master) fix: acil sistem hatasi giderildi
# * bce3464 feat: veritabani baglantisi tamamlandi
# * 3b4322e feat: normal dosya eklendi
# *   b8164de Merge branch 'feature/login' into master (Conflict Resolved)
# |\