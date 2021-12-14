---
title: jdk1.8时间工具类整合
catalog: true
date: 2018/11/20 14:42:52
subtitle: jdk1.8时间工具类整理
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---
## jdk1.8时间工具类整理

```
Java1.8在java.time包下推出了全新的时间日期API
```



```

public class DateUtil {
    private static final Logger log = LoggerFactory.getLogger(DateUtil.class);
    public static final String DATETIME = "yyyy-MM-dd HH:mm:ss";
    public static final String DATE = "yyyy-MM-dd";

    /**
     * 返回hh:MM:ss格式
     * @duration 毫秒
     */
    public static String formatDuration(Integer duration) {
        if (duration == null) {
            return "00:00:00";
        }
        return String.format("%02d", duration / 3600) + ":" + String.format("%02d", (duration % 3600) / 60) + ":" + String.format("%02d", (duration % 3600) % 60);
    }

    /**
     * 返回hh:MM格式
     * @duration 毫秒
     */
    public static String formatDurationOfFourLength(Integer duration) {
        if (duration == null) {
            return "00:00";
        }
        return String.format("%02d", duration / 3600) + ":" + String.format("%02d", (duration % 3600) / 60);
    }


    public static Date getNow() {
        return new Date();
    }

    public static Integer getYear() {
        return LocalDate.now().getYear();
    }

    public static Integer getMonth() {
        return LocalDate.now().getMonth().getValue();
    }

    public static Integer getDayOfWeek() {
        return LocalDate.now().getDayOfWeek().getValue();
    }

    public static Integer getDayOfMonth() {
        return LocalDate.now().getDayOfMonth();
    }

    public static Integer getYear(Date date) {
        return dateToLocalDate(date).getYear();
    }

    public static Integer getMonth(Date date) {
        return dateToLocalDate(date).getMonth().getValue();
    }

    public static Integer getDayOfWeek(Date date) {
        return dateToLocalDate(date).getDayOfWeek().getValue();
    }

    public static Integer getDayOfMonth(Date date) {
        return dateToLocalDate(date).getDayOfMonth();
    }

    public static String format(String pattern) {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(pattern));
    }

    public static String format(Date date, String pattern) {
        if (date == null) {
            return StringUtils.EMPTY;
        }
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTime.format(DateTimeFormatter.ofPattern(pattern));
    }

    public static Date parseDateTime(String dateTime, String pattern) {
        LocalDateTime localDateTime = LocalDateTime.parse(dateTime, DateTimeFormatter.ofPattern(pattern));
        return localDateTimeToDate(localDateTime);
    }

    public static Date parseDate(String date, String pattern) {
        LocalDate dateTime = LocalDate.parse(date, DateTimeFormatter.ofPattern(pattern));
        return localDateToDate(dateTime);
    }

    public static Date plusYears(Date date, long years) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusYears(years));
    }

    public static Date plusMonths(Date date, long months) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusMonths(months));
    }

    public static Date plusDays(Date date, long days) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusDays(days));
    }

    public static Date plusHours(Date date, long hours) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusHours(hours));
    }

    public static Date plusWeeks(Date date, long weeks) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusWeeks(weeks));
    }

    public static Date plusMinutes(Date date, long minutes) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusMinutes(minutes));
    }

    public static Date plusSeconds(Date date, long seconds) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusSeconds(seconds));
    }

    public static Date plusNanos(Date date, long nanos) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.plusNanos(nanos));
    }


    public static Date minusYears(Date date, long years) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusYears(years));
    }

    public static Date minusMonths(Date date, long months) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusMonths(months));
    }

    public static Date minusDays(Date date, long days) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusDays(days));
    }

    public static Date minusHours(Date date, long hours) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusHours(hours));
    }

    public static Date minusWeeks(Date date, long weeks) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusWeeks(weeks));
    }

    public static Date minusMinutes(Date date, long minutes) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusMinutes(minutes));
    }

    public static Date minusSeconds(Date date, long seconds) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusSeconds(seconds));
    }

    public static Date minusNanos(Date date, long nanos) {
        LocalDateTime localDateTime = dateToLocalDateTime(date);
        return localDateTimeToDate(localDateTime.minusNanos(nanos));
    }


    public static Integer betweenToYears(Date var1, Date var2) {
        LocalDate localDate1 = dateToLocalDate(var1);
        LocalDate localDate2 = dateToLocalDate(var2);
        Period period = Period.between(localDate1, localDate2);
        return period.getYears();
    }

    public static Integer betweenToMonths(Date var1, Date var2) {
        return betweenToYears(var1, var2) * 12;
    }

    public static Long betweenToDays(Date var1, Date var2) {
        LocalDate localDate1 = dateToLocalDate(var1);
        LocalDate localDate2 = dateToLocalDate(var2);
        return localDate1.toEpochDay() - localDate2.toEpochDay();
    }

    public static Long betweenToHours(Date var1, Date var2) {
        return betweenToDays(var1, var2) * 24;
    }

    public static Long betweenToMinutes(Date var1, Date var2) {
        return betweenToHours(var1, var2) * 60;
    }

    public static Long betweenToSeconds(Date var1, Date var2) {
        return betweenToMinutes(var1, var2) * 60;
    }

    public static Long betweenToMillis(Date var1, Date var2) {
        return betweenToSeconds(var1, var2) * 1000;
    }

    private static LocalDate dateToLocalDate(Date date) {
        Instant instant = date.toInstant();
        ZoneId zoneId = ZoneId.systemDefault();
        return instant.atZone(zoneId).toLocalDate();
    }

    private static LocalDateTime dateToLocalDateTime(Date date) {
        Instant instant = date.toInstant();
        ZoneId zoneId = ZoneId.systemDefault();
        return instant.atZone(zoneId).toLocalDateTime();
    }

    private static Date localDateToDate(LocalDate localDate) {
        ZoneId zoneId = ZoneId.systemDefault();
        Instant instant = localDate.atStartOfDay().atZone(zoneId).toInstant();
        return Date.from(instant);
    }

    private static Date localDateTimeToDate(LocalDateTime localDateTime) {
        ZoneId zoneId = ZoneId.systemDefault();
        Instant instant = localDateTime.atZone(zoneId).toInstant();
        return Date.from(instant);
    }
}


```
