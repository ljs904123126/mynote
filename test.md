```java
 public static  <T>  Map<Object,T> getMap(List<T> list ,String key){

        Map<Object,T> rmap = new HashMap<>();
        Method readMethod = null;
        for (T t : list) {

            try {
                if(readMethod == null){
                    //Field field = t.getClass().getField(key);
                    PropertyDescriptor pd = new PropertyDescriptor(key,t.getClass());
                    readMethod = pd.getReadMethod();

                }
                 Object invoke = readMethod.invoke(t);
                rmap.put(invoke,t);

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return rmap;
    }
```

