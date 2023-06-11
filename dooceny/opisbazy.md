## Biblioteka studencka

Ten projekt to była bardzo emocjonująca wędrówka przez meandry postgresa. Ja (Szymon Szulc) zająłem się bazą danych. Dzielnie walczyłem o jej spójność. Spędziłem z nią tyle czasu, że udzieliła mi się proza z książek.

Wracając do rzeczy. Zacznę od Pani uwag. Do wszystkich się odnieśliśmy i zamodelowaliśmy związki wiele-do-wielu, usunęliśmy redundancję z tabeli studenci, dodaliśmy do niej loginy. Każda książka jest teraz traktowana osobno, wyróżnikiem grupy tych samych książek jest unikalny token (a la numer ISBN).  Typy kar i etaty zostały przeniesione do osobnych tabel.

Postanowiłem zachować moje szczątkowe komentarze w pliku create.sql, właśnie na nich oparta będzie dalsza część opisu.

Po kolei przejdę po tabelach (tych ciekawszych):

- wydarzenia – nic szczególnego się tu nie dzieje i nie ma co się psuć, postgres na poziomie struktury zapewnił nam sporą spójność. Od siebie dodałem funkcję użytkową, która usuwa przestarzałe wydarzenia.
- oddziały – bardzo skomplikowane usuwanie przez gęstą sieć połączeń kluczami obcymi, potrzebny był wyzwalacz, która najpierw ”oderwie” oddział od reszty i dopiero usunie. Uznałem, że książki zostaną przeniesione do oddziału, gdzie jest ich najmniej. 
- hasła – problematyczne usuwanie, nie mogłem go zabronić, ponieważ użytkownik mógłby chcieć usunąć konto.
- pracownicy i studenci – tu konieczny był wyzwalacz, żeby sprawdzać unikalność nazw użytkowników w 2 rozłącznych tabelach. Usuwanie studenta jest opatrzone wyzwalaczem, który sprawdza, czy wszystkie książki są oddane. Usuwanie pracownika znowu jest problematyczne przez klucze obce, dlatego rozwiązałem to tworząc abstrakcyjnego pracownika ”postgres”, jeśli on znajdzie się, w którejś tabeli to znaczy, że wcześniejszy pracownik został zwolniony.  
- wszystkie tabele związane z książkami – przez unikalny token, który dla postgresa nie jest unikalny, musiałem sam zadbać o poprawne referencjonowanie tokenami, stąd w tej kategorii jest tak dużo wyzwalaczy. Dodatkowo sprawdzam spójność między datą urodzenia autora 
  a wydaniem książki. Na diagramie baza jest niespójna, mam tego świadomość, ale wyzwalacze to naprawiają.
- wypożyczenia, zwroty i kary – ponownie dużo wyzwalaczy i sprawdzania, czy można wypożyczyć książkę. Jeśli tak to następuje stosowna modyfikacja w tabeli książki. Inserty na zwrotach również modyfikują tabele książki, dodatkowo automatycznie nakładają karę za przetrzymanie książki.  

Zdecydowałem się utworzyć 5 indexów (wszędzie na unikalnych tokenach, w wypożyczeniach na książkach i studentach). Inne, o których myślałem zostały automatycznie dodane przez klucz podstawowy.

Na potrzeby aplikacji dodałem 3 widoki, które pomagają w filtrowaniu książek (prosty join i grupowanie po unikalnym tokenie) i wyświetlaniu stanu konta użytkownika (wypożyczenia i zwroty).






