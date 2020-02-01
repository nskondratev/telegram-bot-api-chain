# telegram-bot-api-chain
Middlewares chain for telegram-bot-api package: https://github.com/go-telegram-bot-api/telegram-bot-api

## Motivation
I like the concept of [handler](https://github.com/golang/go/blob/master/src/net/http/server.go#L86) in standard Go http package.
Also I like the middleware chaining, provided by [alice](https://github.com/justinas/alice) package.
 
This package introduces Handler and HandlerFunc types for telegram-bot-api package and utility functions for chaining middleware for updates handler.

## Example
```go
package main

import (
    "context"
    "log"
    "os"
    "time"

    "github.com/go-telegram-bot-api/telegram-bot-api"
    chain "github.com/nskondratev/telegram-bot-api-chain"
)

func LogTimeExecution(next chain.Handler) chain.Handler {
	return bot.HandlerFunc(func(ctx context.Context, update tgbotapi.Update) {
		ts := time.Now()
		next.Handle(ctx, update)
		te := time.Now().Sub(ts)
        log.Printf("Execution time of handler: %s", te.String())
	})
}

func UpdateHandler(ctx context.Context, update tgbotapi.Update) {
    if update.Message != nil && update.Message.From != nil {
    	log.Printf("Received update from: %s\n", update.Message.From)
    }
}

func main() {
    h := chain.
        NewChain(
            LogTimeExecution,
        ).
        ThenFunc(chain.HandlerFunc(UpdateHandler))

    tg, _ := tgbotapi.NewBotAPI(os.Getenv("TELEGRAM_API_TOKEN"))
    updateConfig := tgbotapi.NewUpdate(0)
    updateConfig.Timeout = 60
    
    updates, _ := tg.GetUpdatesChan(updateConfig)
    
    for u := range updates {
    	h.Handle(context.Background(), u)
    }
}
```
