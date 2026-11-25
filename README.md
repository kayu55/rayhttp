# rayhttp
## babi
````
cat > /usr/bin/babi << 'EOF'
#!/bin/bash
CFG=/etc/xray/config.json
LOG=/var/log/xray/access.log
HOLD_FILE=/tmp/babi_hold
touch $HOLD_FILE
while true; do
clear
echo "=== BABI v12 - JENDELA 120 DETIK | HOLD 30 DETIK ==="
echo "Mentah = itungan saiki | Ditahan = sing ditampilno"
echo "Tekan ENTER gawe balik menu"
NOW=$(date +%s)
NO=0
for akun in $(grep '###' "$CFG" | awk '{print $2}' | sort | uniq); do
  LAST_LINE=$(grep -w "email: $akun" "$LOG" 2>/dev/null | tail -n 1)
  [ -z "$LAST_LINE" ] && continue
  LAST_EPOCH=$(echo "$LAST_LINE" | awk '{split($1,a,"/"); split($2,b,":"); split(b[3],c,"."); print mktime(a[1]" "a[2]" "a[3]" "b[1]" "b[2]" "c[1])}')
  DIFF=$(( NOW - LAST_EPOCH ))
  if [[ $DIFF -gt 15 ]]; then
    sed -i "/^$akun /d" $HOLD_FILE
    continue
  fi
  CNT120=$(grep -w "email: $akun" "$LOG" 2>/dev/null | tail -n 8000 | awk -v now=$NOW '{split($1,a,"/"); split($2,b,":"); split(b[3],c,"."); ep=mktime(a[1]" "a[2]" "a[3]" "b[1]" "b[2]" "c[1]); if(now-ep<=120 && now-ep>=-5) cnt++;} END{print (cnt+0)}')
  EST=$(( (CNT120 + 35) / 70 ))
  if [[ $EST -lt 1 ]]; then EST=1; fi
  OLD=$(grep "^$akun " $HOLD_FILE | awk '{print $2}')
  OLD_TIME=$(grep "^$akun " $HOLD_FILE | awk '{print $3}')
  [ -z "$OLD" ] && OLD=$EST && OLD_TIME=$NOW
  if [[ $EST -ne $OLD ]]; then
    if [[ $((NOW - OLD_TIME)) -gt 30 ]]; then
      FINAL=$EST
      sed -i "/^$akun /d" $HOLD_FILE
      echo "$akun $FINAL $NOW" >> $HOLD_FILE
    else
      FINAL=$OLD
    fi
  else
    FINAL=$EST
    sed -i "/^$akun /d" $HOLD_FILE
    echo "$akun $FINAL $NOW" >> $HOLD_FILE
  fi
  LAST=$(echo "$LAST_LINE" | awk '{print $1" "$2}')
  NO=$((NO + 1))
  echo ""
  echo "$NO. user : $akun"
  echo "   last : $LAST (aktif $DIFF detik lalu)"
  echo "   trafik 120 detik : $CNT120 line | mentah : $EST HP | ditahan : $FINAL HP"
  echo "   => Estimasi : $FINAL HP"
done
echo ""
echo "=========================================="
echo "Total online : $NO user"
echo "Tekan [ENTER] balik menu..."
if read -t 3 -rsn1 key; then
  break
fi
done
clear
menu
EOF
chmod +x /usr/bin/babi
rm -f /tmp/babi_hold
babi
````
