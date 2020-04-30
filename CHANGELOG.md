# Changelog

## Version 1.3.1
- Apply suggestion from issue #24

## Version 1.3.0
- Updated API to 4.8

## Version 1.3.0-beta1
- Updated API to 4.7

## Version 1.3.0-beta
Added new feature - Chain. It is common case when you need to ask the user several
questions sequentially and process user errors. Now you can create such chains easily.
Sea the example below. Do not forget to call `build()` method at the end =)

```kotlin
fun main() {
    val token = "<TOKEN>"
    val username = "<BOT USERNAME>"
    val bot = Bot.createPolling(username, token)

    bot.chain("/start") { msg -> bot.sendMessage(msg.chat.id, "Hi! What is your name?") }
        .then { msg -> bot.sendMessage(msg.chat.id, "Nice to meet you, ${msg.text}! Send something to me") }
        .then { msg -> bot.sendMessage(msg.chat.id, "Fine! See you soon") }
        .build()

    bot.chain(
        label = "location_chain",
        predicate = { msg -> msg.location != null },
        action = { msg ->
            bot.sendMessage(
                msg.chat.id,
                "Fine, u've sent me a location. Is this where you want to order a taxi?(yes|no)"
            )
        })
        .then("answer_choice") { msg ->
            when (msg.text) {
                "yes" -> bot.jumpToAndFire("order_taxi", msg)
                "no" -> bot.jumpToAndFire("cancel_ordering", msg)
                else -> {
                    bot.sendMessage(msg.chat.id, "Oops, I don't understand you. Just answer yes or no?")
                    bot.jumpTo("answer_choice", msg)
                }
            }
        }
        .then("order_taxi", isTerminal = true) { msg -> 
            bot.sendMessage(msg.chat.id, "Fine! Taxi is coming") 
        }
        .then("cancel_ordering", isTerminal = true) { msg -> 
            bot.sendMessage(msg.chat.id, "Ok! See you next time") 
        }
        .build()

    bot.start()
}
```