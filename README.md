import cn.hutool.core.codec.Base64;
import cn.hutool.core.util.CharsetUtil;
import cn.hutool.core.util.HexUtil;
import cn.hutool.crypto.symmetric.SymmetricAlgorithm;
import cn.hutool.crypto.symmetric.SymmetricCrypto;

public class AesUtils {

    public static String encode(String data, String apiKey) {
        SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, HexUtil.decodeHex(apiKey));
        return aes.encryptHex(Base64.encode(data),CharsetUtil.CHARSET_UTF_8);
    }

    public static String decode(String encodeData, String apiKey) {
        SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, HexUtil.decodeHex(apiKey));
        return Base64.decodeStr(aes.decryptStr(encodeData,CharsetUtil.CHARSET_UTF_8));
    }

    public static void main(String[] args) {

        String apiKey = "a2450a85a4b00b626ed94490101a8613";

        String data = "requestNo=123&currency=USD&limitAmount=100&cardType=NORMAL";

        System.out.println("原文：" + data);
        String encodeData = encode(data, apiKey);
        System.out.println("加密后：" + encodeData);

        String decodeData = decode(encodeData, apiKey);
        System.out.println("解密后：" + decodeData);

        System.out.println("解密后与原文比较：" + data.equals(decodeData));
    }

}
