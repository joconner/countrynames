# Enabling CLDR Locale Data in Java and Groovy

Today a colleague asked whether I could provide a list of localized country names for Portuguese. I know that the
[CLDR](http://cldr.unicode.org/) database contains all that great localization goodness, but I didn't have the time to dig through the raw data. Still, I wanted to help, and I know that the Java platform has included CLDR data since 1.8. I decided to create a short Groovy language script to retrieve and display the information. Because Groovy runs in the JVM, it also has access to that CLDR information.

## The Groovy Script

Here's the script I came up with. I named it "countries".

    #!/usr/bin/env groovy
    /**
     * Created by joconner on 6/29/16.
     */

    supportedLangs = Locale.getISOLanguages();

    if (!(this.args.length > 0 && supportedLangs.contains(this.args[0].toLowerCase()))) {
        printf("Usage: countries <lang_code>\n")
        printf("\t<lang_code> must be an ISO lang code. Examples: en, pt, fr, es, ja.\n")
        System.exit(0)
    }
    currentLocale = new Locale.Builder().setLanguageTag(this.args[0].toLowerCase()).build()
    locales = Locale.getISOCountries()
    for(c in locales) {
        targetLocale = new Locale.Builder().setRegion(c).build()
        printf("%s, %s\n", targetLocale.getCountry(), targetLocale.getDisplayCountry(currentLocale))
    }

I won't go into the script in detail. If you are familiar with Java, you've probably seen these classes and methods before. The interesting thing is that *the CLDR data is not enabled by default*.

## Enabling CLDR

For backward compatibility, CLDR locale data is not enabled by default. However, you can enable it by setting a system property on the Java command line. The system property is [_java.locale.providers_](https://docs.oracle.com/javase/8/docs/technotes/guides/intl/enhancements.8.html#cldr). If you set this to the value _CLDR_, the JRE will use CLDR as the locale data source.

Run the above script like this to get Portuguese (pt) names for all supported ISO countries in the JRE:
    groovy -Djava.locale.providers=CLDR countries pt

Although this example is written in Groovy, the same system property is available with the Java command line as well. Just run your Java application like this:
    java -Djava.locale.providers=CLDR <yourapphere>

## Analyzing Differences

I ran the above script twice, once using CLDR data and once using the default JRE data. For the _pt_ language, I discovered a couple differences in JDK 1.8.0_92:

<table border="1">
<tr><th>CLDR</th><th>JRE</th></tr>
<tr><td>BQ, Bonaire, Sint Eustatius, and Saba</td><td>BQ, Bonaire, Sint Eustatius and Saba</td></tr>
<tr><td>SX, Sint Maarten</td><td>SX, Sint Maarten (Dutch part)</td></tr>
</table>

I suspect you'll find other and perhaps more significant differences for other languages.

## Downloading Code

You can git the code for this article from my [github account](https://github.com/joconner). Enjoy!