# Formato de Código — Ruby on Rails

Complementa las [reglas de formato comunes](./README.md). Se aplica a todos los proyectos Ruby on Rails de la organización.

---

## Herramienta principal: RuboCop

El formateo y análisis estático se realiza con [RuboCop](https://rubocop.org/) y las extensiones relevantes:

```ruby
# Gemfile (grupo development + test)
gem "rubocop", require: false
gem "rubocop-rails", require: false
gem "rubocop-rspec", require: false
gem "rubocop-performance", require: false
```

---

## Configuración base `.rubocop.yml`

```yaml
require:
  - rubocop-rails
  - rubocop-rspec
  - rubocop-performance

AllCops:
  NewCops: enable
  TargetRubyVersion: 3.2
  Exclude:
    - "db/schema.rb"
    - "vendor/**/*"
    - "node_modules/**/*"
    - "bin/**/*"

Layout/LineLength:
  Max: 120

Style/Documentation:
  Enabled: false

Style/FrozenStringLiteralComment:
  Enabled: true
  EnforcedStyle: always
```

---

## Convenciones de naming (Ruby)

| Elemento           | Convención             | Ejemplo                                      |
| ------------------ | ---------------------- | -------------------------------------------- |
| Clases, Módulos    | `CamelCase`            | `OrderService`, `Payments::Processor`        |
| Métodos, variables | `snake_case`           | `get_order_by_id`, `payment_result`          |
| Constantes         | `SCREAMING_SNAKE_CASE` | `MAX_RETRY_COUNT`, `API_VERSION`             |
| Archivos           | `snake_case`           | `order_service.rb`, `charge_card_service.rb` |
| Predicados         | Sufijo `?`             | `active?`, `valid?`, `can_delete?`           |
| Bang methods       | Sufijo `!`             | `save!`, `update!`, `destroy!`               |

---

## Estilos de código preferidos

```ruby
# ✅ Usar guard clauses para evitar anidación excesiva
def process_order(order)
  return Result.error("Order not found") unless order
  return Result.error("Order already processed") if order.processed?

  order.process!
  Result.success(order)
end

# ✅ Métodos de una línea con do/end para bloques multi-línea
orders.each do |order|
  order.update!(status: :processed)
end

# ✅ Symbol to proc para transformaciones simples
names = users.map(&:name)

# ✅ Hash rocket solo para claves no-símbolo
config = { "Content-Type" => "application/json" }

# ✅ Nuevos hashes con dos puntos
config = { timeout: 30, retries: 3 }
```

---

## Organización de código Rails

```text
app/
├── controllers/         # Solo routing + serialización; sin lógica de negocio
├── models/              # ActiveRecord + validaciones + scopes
├── services/            # Casos de uso (Plain Old Ruby Objects)
│   └── payments/
│       ├── charge_card_service.rb
│       └── refund_service.rb
├── queries/             # Queries complejas aisladas de los modelos
├── policies/            # Autorización (Pundit recomendado)
├── serializers/         # Presentación de respuestas JSON (jsonapi-serializer)
├── jobs/                # Background jobs (Sidekiq / ActiveJob)
├── mailers/             # Emails
└── helpers/             # Helpers de vistas (solo para vistas)
```

---

## Services Objects

Los service objects son la pieza central de la lógica de negocio en Rails:

```ruby
# app/services/payments/charge_card_service.rb
module Payments
  class ChargeCardService
    def initialize(order:, payment_method:)
      @order = order
      @payment_method = payment_method
    end

    def call
      # validations + logic
      Result.success(charge)
    rescue PaymentGateway::Error => e
      Result.failure(e.message)
    end

    private

    attr_reader :order, :payment_method
  end
end

# Uso
result = Payments::ChargeCardService.new(order: order, payment_method: card).call
```

---

## Tests con RSpec

```ruby
# spec/services/payments/charge_card_service_spec.rb
RSpec.describe Payments::ChargeCardService do
  describe "#call" do
    context "when the payment is successful" do
      it "returns a successful result" do
        # Arrange
        order = build(:order)
        card = build(:payment_method)

        # Act
        result = described_class.new(order: order, payment_method: card).call

        # Assert
        expect(result).to be_success
      end
    end
  end
end
```

---

## Verificación en CI

```sh
# Verificar formato y estilo
bundle exec rubocop

# Solo verificar sin corregir (modo CI)
bundle exec rubocop --no-color --format progress

# Corregir automáticamente lo que sea seguro
bundle exec rubocop --autocorrect-all
```

Ver workflow completo en [CI/CD — Ruby on Rails](../ci-cd/ruby-on-rails.md).
