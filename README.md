# ==========================================
# 1. SYSTEM IDENTITY & USER
# ==========================================

# ==========================================
# 2. NTP & DNS
# ==========================================

# ==========================================
# 3. LAN BRIDGE & IP (ether3,4,5 assumed as LAN)
# ==========================================

# ==========================================
# 4. DHCP SERVER
# ==========================================

# ==========================================
# 5. NAT (MASQUERADE FOR BOTH WANs)
# ==========================================

# ==========================================
# 6. LOAD BALANCING & FAILOVER (PCC + ROUTES)

# ==========================================
# 7. TELEGRAM WAN MONITORING SCRIPT
# ==========================================

# 8. SCHEDULER (RUN EVERY 10 SECONDS)
# ==========================================


Pontos importantes:
Regra padrão: Em muitas configurações padrão do MikroTik, a porta 53 já pode estar bloqueada, mas é crucial verificar se a opção IP > DNS > Allow Remote Requests está desmarcada 
forum.mikrotik.com
forum.mikrotik.com
.
DNS over HTTPS/TLS: Bloquear apenas a porta 53 não impede DoH (porta 443) ou DoT (porta 853). Se quiser bloquear também, inclua a porta 853 nas regras 
forum.mikrotik.com
.
Teste de segurança: Após configurar, use ferramentas como nmap ou verificadores online para confirmar que a porta 53 não está acessível publicamente.
Não afeta a LAN: Essas regras na interface WAN não interferem nas requisições DNS dos seus dispositivos internos, que continuarão funcionando normalmente.
Se quiser, posso ajudar a revisar sua configuração atual ou tirar dúvidas sobre outras regras de firewall. É só me dizer o que precisa! 😊
