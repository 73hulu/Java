# Gson

```java
public class JsonKit implements ExclusionStrategy{
    String[] keys;

    public JsonKit(String[] keys) {
        this.keys = keys;
    }

    @Override
    public boolean shouldSkipField(FieldAttributes arg0) {
        for (String key : keys) {
            if (key.equals(arg0.getName())) {
                return true;
            }
        }
        return false;
    }

    @Override
    public boolean shouldSkipClass(Class<?> clazz) {
        return false;
    }
}

```
