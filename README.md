## a. How much data will your publisher send to the message broker in one run?

Publisher mengirimkan 5 event dalam satu kali run. Setiap event berisi objek `UserCreatedEventMessage` yang memiliki dua field, yaitu `user_id` (String) dan `user_name` (String). Dengan demikian, total terdapat 5 pesan yang dikirim ke queue bernama `user_created`.

## b. The URL `amqp://guest:guest@localhost:5672` is the same as in subscriber. What does it mean?

URL yang sama menunjukkan bahwa publisher dan subscriber terhubung ke message broker yang sama, yaitu RabbitMQ yang berjalan pada `localhost` dengan port `5672`. Hal ini merupakan inti dari event-driven architecture, di mana publisher dan subscriber tidak saling berkomunikasi secara langsung, melainkan melalui broker sebagai perantara. Publisher mengirim pesan ke broker, sedangkan subscriber membaca pesan dari broker yang sama.