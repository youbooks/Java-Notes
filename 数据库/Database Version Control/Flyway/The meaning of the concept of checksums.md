# The meaning of the concept of checksums

Checksum field in Flyway forming a part of **verification mechanism** ensuring that migration scripts haven't been changed since they applied to the database. This will guaranty that all instances of your application will have same database structure (content). You can switch off verification, but I will not recommend you to do so. Answering you questions:

> What is?

Just google what is checksum. [Wikipedia](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)

> How is it calculated?

For SQL migrations Flyway uses CRC32 class to calculate the checksum. For exact code see below.

> How can it be changed?

**The checksum of the migration will be changed once the binary content of you migration modified**. If you want to change checksum field in the DB when you need to calculate the checksum for the new version of your migration file and then change the value in the DB. However, I wouldn't recommend to do that. You shouldn't need to do that and the fact that you want to change it may indicate that you doing something wrong. Anyway, the calculation code for the checksum is quite simple (with courtesy of Flyway source code):

```java
/**
         * Calculates the checksum of this string.
         *
         * @param str The string to calculate the checksum for.
         * @return The crc-32 checksum of the bytes.
         */
        /* private -> for testing */
        static int calculateChecksum(Resource resource, String str) {
            final CRC32 crc32 = new CRC32();

            BufferedReader bufferedReader = new BufferedReader(new StringReader(str));
            try {
                String line;
                while ((line = bufferedReader.readLine()) != null) {
                    crc32.update(line.getBytes("UTF-8"));
                }
            } catch (IOException e) {
                String message = "Unable to calculate checksum";
                if (resource != null) {
                    message += " for " + resource.getLocation() + " (" + resource.getLocationOnDisk() + ")";
                }
                throw new FlywayException(message, e);
            }

            return (int) crc32.getValue();
        }
```

