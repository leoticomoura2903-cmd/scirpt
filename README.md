
const KEY_SYSTEM = {
    // Banco de dados de keys
    keys: {
        "1DIA": {},
        "7DIAS": {},
        "30DIAS": {}
    },
    
    // Gerar key com validade personalizada
    generateKey(dias, level = "USER") {
        const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
        let code = '';
        for (let i = 0; i < 6; i++) {
            code += chars.charAt(Math.floor(Math.random() * chars.length));
        }
        
        const now = new Date();
        const expireTime = new Date(now);
        expireTime.setDate(expireTime.getDate() + dias);
        
        // Define qual banco usar
        let db;
        if (dias === 1) db = this.keys["1DIA"];
        else if (dias === 7) db = this.keys["7DIAS"];
        else if (dias === 30) db = this.keys["30DIAS"];
        else return null;
        
        const prefix = dias === 1 ? "1DIA" : dias === 7 ? "7DIAS" : "30DIAS";
        const key = `XIT-${prefix}-${code}`;
        
        db[key] = {
            expires: expireTime,
            created: now,
            level: level,
            duration: `${dias} dias`
        };
        
        console.log(`✅ KEY ${dias} DIAS GERADA: ${key}`);
        console.log(`⏰ Expira em: ${expireTime.toLocaleString()}`);
        console.log(`👤 Nível: ${level}`);
        
        return key;
    },
    
    // Validar qualquer key
    validateKey(inputKey) {
        // Procura em todos os bancos
        for (const dbName in this.keys) {
            const db = this.keys[dbName];
            if (db[inputKey]) {
                const entry = db[inputKey];
                const now = new Date();
                
                if (now > entry.expires) {
                    delete db[inputKey];
                    return { valid: false, reason: `❌ Chave expirada (${entry.duration})` };
                }
                
                const diffMs = entry.expires - now;
                const diffDays = Math.ceil(diffMs / (1000 * 60 * 60 * 24));
                const diffHours = Math.ceil(diffMs / (1000 * 60 * 60));
                
                return {
                    valid: true,
                    level: entry.level,
                    daysLeft: diffDays,
                    hoursLeft: diffHours,
                    duration: entry.duration,
                    reason: `✅ Válida! ${diffDays} dias restantes`
                };
            }
        }
        return { valid: false, reason: "❌ Chave não encontrada" };
    },
    
    // Listar todas as keys
    listKeys() {
        console.log("📋 KEYS ATIVAS:");
        for (const dbName in this.keys) {
            const db = this.keys[dbName];
            const count = Object.keys(db).length;
            console.log(`   ${dbName}: ${count} keys`);
        }
    }
};

// 1. Gerar keys
const key1 = KEY_SYSTEM.generateKey(1, "VIP");      // 1 dia
const key2 = KEY_SYSTEM.generateKey(7, "ADMIN");    // 7 dias
const key3 = KEY_SYSTEM.generateKey(30, "MASTER");  // 30 dias

// 2. Validar keys
console.log(KEY_SYSTEM.validateKey(key1));
console.log(KEY_SYSTEM.validateKey(key2));
console.log(KEY_SYSTEM.validateKey(key3));

// 3. Listar keys ativas
KEY_SYSTEM.listKeys();

// 4. Testar key inválida
console.log(KEY_SYSTEM.validateKey("XIT-INVALIDO"));

function disableKey(key) {
    // Procura em todos os bancos
    const databases = [KEYS_1DIA, KEYS_7DIAS, KEYS_30DIAS];
    
    for (const db of databases) {
        if (db && db[key]) {
            // Marca como desativada
            db[key].disabled = true;
            db[key].disabledAt = new Date();
            console.log(`🔒 KEY DESATIVADA: ${key}`);
            console.log(`📅 Desativada em: ${new Date().toLocaleString()}`);
            return true;
        }
    }
    
    console.log(`❌ Key não encontrada: ${key}`);
    return false;
}

// ===== VALIDAR COM VERIFICAÇÃO DE DESATIVAÇÃO =====
function validateKeyWithDisable(inputKey) {
    const databases = [KEYS_1DIA, KEYS_7DIAS, KEYS_30DIAS];
    
    for (const db of databases) {
        if (db && db[inputKey]) {
            const entry = db[inputKey];
            
            // Verifica se está desativada
            if (entry.disabled) {
                return {
                    valid: false,
                    reason: `🔒 Key desativada em ${entry.disabledAt?.toLocaleString() || 'data desconhecida'}`,
                    disabled: true
                };
            }
            
            // Verifica se expirou
            const now = new Date();
            if (now > entry.expires) {
                delete db[inputKey];
                return { valid: false, reason: "❌ Chave expirada" };
            }
            
            const diffMs = entry.expires - now;
            const diffDays = Math.ceil(diffMs / (1000 * 60 * 60 * 24));
            
            return {
                valid: true,
                level: entry.level,
                daysLeft: diffDays,
                reason: `✅ Válida! ${diffDays} dias restantes`
            };
        }
    }
    return { valid: false, reason: "❌ Chave não encontrada" };
}

// ===== EXEMPLO =====
const minhaKey = generateKey1Dia("VIP");
console.log(validateKeyWithDisable(minhaKey)); // ✅ Válida
disableKey(minhaKey); // 🔒 Desativa
console.log(validateKeyWithDisable(minhaKey)); // 🔒 Key desativada
