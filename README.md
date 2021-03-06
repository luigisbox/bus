# Bus

Extremely lightweight message bus without mixins, globals or mutable state.

[![Build Status](https://travis-ci.org/minio-sk/bus.png)](https://travis-ci.org/minio-sk/bus) [![Code Climate](https://codeclimate.com/github/minio-sk/bus.png)](https://codeclimate.com/github/minio-sk/bus) [![Code coverage](https://codeclimate.com/github/minio-sk/bus/coverage.png)](https://codeclimate.com/github/minio-sk/bus)

## Installation

Add this line to your application's Gemfile:

    gem 'bus'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install bus

## Usage

```ruby
# global config or initializer or application controller
bus = Bus.new([
  DonationReceipt,
  RealtimeViewerUpdate
])


# controller
class DonationsController # < ActionController::Base
  def create
    listener = bus.
      on(:donation_created) {|donation| send_user_to_payment_gateway(donation) }. # NOTE the dot!
      on(:donation_invalid) {|donation| render donation.errors, status: :bad_request }

    # or

    listener = bus.on(
      donation_created: ->(donation) { send_user_to_payment_gateway(donation) },
      donation_invalid: ->(donation) { render donation.errors, status: :bad_request }
    )

    DonationService.new.add_donation(current_user, params[:donation], listener)
    
    # or
    
    DonationService.new.add_donation(current_user, params[:donation], bus.when(
      donation_created: ->(donation) { send_user_to_payment_gateway(donation) },
      donation_invalid: ->(donation) { render donation.errors, status: :bad_request }
    ))
  end

  private
  def send_user_to_payment_gateway(donation)
    #...
  end
end


# service
class DonationService
  def add_donation(donor, donation_attributes, listener)
    donation = donor.donations.build(donation_attributes)
    if donation.save
      listener.donation_created(donation)
    else
      listener.donation_invalid(donation)
    end
  end
end

# listeners
class DonationReceipt
  def donation_created(donation)
    Mailer.delay.send_donation_receipt(donation)
  end
end

class RealtimeViewerUpdate
  def donation_created(donation)
    #..
  end
end
```

## Contributing

1. Fork it ( http://github.com/minio-sk/bus/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
