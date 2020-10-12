# WeatherFromGL
#ver_1.0.0





















object KeyStoreUtil {
    private const val alias = "ALIAS"
    private var keyStore: KeyStore = KeyStore.getInstance("AndroidKeyStore").also { it.load(null) }
    private const val CIPHER_TRANSFORMATION = "RSA/ECB/PKCS1Padding"
    init {
        if (!containsAlias()) {
            getKeyPair()
        }
    }

    private fun getKeyPair() {
        val kpg = KeyPairGenerator.getInstance(
            KeyProperties.KEY_ALGORITHM_RSA, "AndroidKeyStore"
        )
        kpg.initialize(
            KeyGenParameterSpec.Builder(
                alias,
                KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
            )
                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_RSA_PKCS1)
                .setDigests(
                    KeyProperties.DIGEST_SHA256, KeyProperties.DIGEST_SHA512
                ).build()
        )
        kpg.generateKeyPair()
    }

    fun encode(content: String?): ByteArray? {
        if (content == null) {
            return null
        }
        try {
            val publicKey = keyStore.getCertificate(alias).publicKey
            val cipher = Cipher.getInstance(CIPHER_TRANSFORMATION)
            cipher.init(Cipher.ENCRYPT_MODE, publicKey)
            return cipher.doFinal(content.toByteArray())
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return null
    }

    fun decode(data: ByteArray?): String? {
        if (data == null) {
            return null
        }
        try {
            val entry = keyStore.getEntry(alias, null)
            val privateKeyEntry = entry as KeyStore.PrivateKeyEntry
            val cipher = Cipher.getInstance(CIPHER_TRANSFORMATION)
            cipher.init(Cipher.DECRYPT_MODE, privateKeyEntry.privateKey)
            return String(cipher.doFinal(data))
        } catch (e:Exception) {
            e.printStackTrace()
        }
        return null
    }
    
    
    private fun containsAlias(): Boolean {
        if (TextUtils.isEmpty(alias)) {
            return false
        }
        var contains = false
        try {
            contains = keyStore.containsAlias(alias)
        } catch (e: java.lang.Exception) {
            e.printStackTrace()
        }
        return contains
    }
    }

