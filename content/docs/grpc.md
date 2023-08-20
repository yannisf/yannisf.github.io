---
title: "grpcurl"
date: 2023-08-17T16:06:40+03:00
---

* Use grpcurl to test your service

```bash
$ grpcurl --plaintext localhost:6565 list
eu.frlab.diceware.proto.Diceware
grpc.health.v1.Health
grpc.reflection.v1alpha.ServerReflection

$ grpcurl --plaintext localhost:6565 list eu.frlab.diceware.proto.Diceware
eu.frlab.diceware.proto.Diceware.roll

$ grpcurl --plaintext localhost:6565 describe eu.frlab.diceware.proto.Diceware.roll
eu.frlab.diceware.proto.Diceware.roll is a method:
rpc roll ( .eu.frlab.diceware.proto.DicewareRequest ) returns ( .eu.frlab.diceware.proto.DicewareResponse );

$ grpcurl --plaintext localhost:6565 describe .eu.frlab.diceware.proto.DicewareRequest
eu.frlab.diceware.proto.DicewareRequest is a message:
message DicewareRequest {
  bool shortCodes = 1;
  int32 numberOfWords = 2;
  .eu.frlab.diceware.proto.DicewareRequest.ConcatMode concatMode = 3;
  enum ConcatMode {
    SIMPLE = 0;
    SPACE = 1;
    CAMEL = 2;
    PASCAL = 3;
    SNAKE = 4;
    KEBAB = 5;
  }
}

$ grpcurl --plaintext -d '{"shortCodes":true, "numberOfWords":3, "concatMode":0}' localhost:6565 eu.frlab.diceware.proto.Diceware.roll
{
  "password": "wifi harm click",
  "codeWordPairs": [
    {
      "code": 6555,
      "word": "wifi"
    },
    {
      "code": 3363,
      "word": "harm"
    },
    {
      "code": 1645,
      "word": "click"
    }
  ]
}
```
