# DEBUGGING – BookingSystem

## Hur jag arbetade

Jag började i Visual Studio med att köra testerna i Test Explorer. När ett test var rött läste jag testets namn och vad det förväntade sig. Sedan använde jag Ctrl + klick för att följa metoderna till rätt kod. Jag försökte förstå felet, tog hjälp av AI för att få koden förklarad, ändrade koden och körde testerna igen.

## Felen i den ordning jag arbetade

**1. Skapa en giltig bokning.** Jag började med `Create_WhenBookingIsValid_AddsAndSavesBooking()`. Från testet följde jag `Create()` och sedan `Validate()`. Där undersökte jag kontrollen av starttid och sluttid. Kontrollen behövde neka en bokning om starttiden var lika med eller senare än sluttiden. Villkoret blev `booking.StartTime >= booking.EndTime`. Jag körde testet igen efter ändringen.

**2. Avboka en bokning.** Sedan arbetade jag med `Cancel_WhenBookingExists_CancelsAndSavesBooking()`. I `Cancel()` stod det att funktionen inte var färdig och metoden returnerade bara ett felmeddelande. Jag ändrade den så att den hittar bokningen med ID, sätter `IsCancelled = true` och anropar `repository.Save()`.

**3. Visa aktiva bokningar.** Testet `GetActiveBookings_ReturnsOnlyActiveBookingsSortedByDateAndStartTime()` visade att metoden inte filtrerade eller sorterade bokningarna. Jag lade till `Where()` för att ta bort avbokade bokningar, `OrderBy()` för datum och `ThenBy()` för starttid.

**4. Söka efter kund.** I `Search_WhenSearchTermHasDifferentCasing_ReturnsBooking()` fungerade inte sökning med andra stora och små bokstäver. Jag följde `Search()` och ändrade `Contains(searchTerm)` till `Contains(searchTerm, StringComparison.OrdinalIgnoreCase)`.

**5. Överlappande bokningar – felet jag förklarar mer.** Senare kom jag tillbaka till `Validate()` genom testet `Create_WhenBookingOverlapsExistingBooking_ReturnsFailure()`. Det fanns redan en bokning kl. 09:00–11:00. En ny bokning kl. 08:00–10:00 skulle nekas, men den gamla kontrollen missade den eftersom den bara undersökte om **den nya starttiden** låg inom den gamla bokningen:

```csharp
booking.StartTime >= existing.StartTime &&
booking.StartTime < existing.EndTime
```

Felets orsak var alltså att sluttiden på den nya bokningen inte jämfördes med den befintliga starttiden. Jag ersatte villkoret med:

```csharp
booking.StartTime < existing.EndTime &&
booking.EndTime > existing.StartTime
```

Nu kontrolleras båda tidsintervallen. Jag verifierade ändringen genom att köra testets fyra olika överlappningsfall. En alternativ lösning hade varit flera separata `if`-satser för olika typer av överlappning, men jag valde två jämförelser eftersom de är kortare och täcker fallen utan onödig upprepning. Testet hjälper också till att upptäcka om samma fel kommer tillbaka senare.

**6. Felaktigt deltagarantal.** Till sist testade jag konsolprogrammet manuellt. I `CreateBooking()` kraschade `int.Parse()` om jag skrev `six`. Jag bytte till `int.TryParse()` och visade ett felmeddelande i stället. Jag kontrollerade också att 14:00–12:00 nekades och att en sparad bokning fanns kvar efter omstart.

## Resultat

Efter ändringarna var alla **14 automatiska tester gröna**. Jag lärde mig att läsa testerna först, följa metoderna steg för steg och kontrollera varje ändring innan jag gick vidare.
