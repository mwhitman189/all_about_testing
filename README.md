# All About Testing
This document organizes some disparate knowledge of software testing, particularly for web development.
It includes a brief overview of Test-Driven-Development (TDD), as well as common types of testing, their use cases, and their related frameworks & libraries, with examples.
## [Test-Driven Development](#tdd), and [CI / CD](#continuous-integration-and-continuous-development)
### TDD
TDD is a development process that promotes short development cycles in which reqirements are turned into specific test cases, and then software is refined to pass those tests.
Example:
Specification - Landing page lists items in database
=> Test:
`
def index(self):
    """ Check that get request returns correct number of items in db """
    c = Client()
    response = c.get('/')
    self.assertEqual(response.status_code, 200)
    self.assertEqual(len(response.context["items"]), 5)    
`
This test will pass only if the server responds with a status code of 200, setting a concrete, easily testable goal for developers.

Each time a new feature is added, or a bug is debugged, a new test is created. This ensures robust coverage of the app, while also allowing rapid development.

### Continuous Integration and Continuous Development
CI, or 'Continuous Integration', is a development paradigm which emphasizes frequent merges to a master branch, with automated [unit testing](#unit-testing) to ensure that one commit does not break other part so the application.


## Types of Testing
- [Unit Testing](#unit-testing)
- [Integration Testing](#integration-testing)

### Unit Testing
Testing individual pieces of the software is known as 'Unit Testing', as each piece serves an individual function that can be tested on its own.
- [Back-end Testing](#backend-testing)
- [Browser Testing](#frontend-testing)
#### Back-end Testing
- [Unittest](#unittest)
- [Django.test](#django.test)
##### Unittest
Python provide the Unittest library to support back-end testing
`
import unittest

class Tests(unittest.TestCase):
    def test_1(self):
        """Check that 1 is not prime"""
        self.assertFalse(is_prime_and_greater_than(1, 0))
`
##### Django.test
Django provides some additional utilities.
Here is an example from an auction site project (the first four are for DB testing; the last four are for page functionality):
`
from django.test import TestCase, Client
from django.utils import timezone
from django.db.models import Max

from .models import User, Rating, AuctionListing, WatchList, Bid, Comment

class AuctionTestCase(TestCase):
    """ Unit Tests """

    def setUp(self):
        # Create users
        u1 = User.objects.create(
            username="Ronald", password="MacInDaHouse", is_superuser=True)
        u2 = User.objects.create(username="Dennis", password="GettinDennised")

        # Create listings
        a1 = AuctionListing.objects.create(seller=u1, item="1964 Mustang", category="MTR", expiration=timezone.now(
        ) + timezone.timedelta(days=3), starting_bid=250, buyout_price=300)
        a2 = AuctionListing.objects.create(seller=u2, item="Lego Camper", category="TNH", expiration=timezone.now(
        ) + timezone.timedelta(days=-3), starting_bid=59, buyout_price=199)
        a3 = AuctionListing.objects.create(seller=u2, item="", category="SHFT", expiration=timezone.now(
        ) + timezone.timedelta(days=3), starting_bid=-15, buyout_price=0)
        a4 = AuctionListing.objects.create(seller=u1, item="Superman Action Figure", expiration=timezone.now(
        ) + timezone.timedelta(days=3), category="TNH", starting_bid=50, buyout_price=45)

        # Create bids
        b1 = Bid.objects.create(bidder=u2, listing=a1,
                                amount=300, is_active=True, is_winner=False)
        b2 = Bid.objects.create(bidder=u2, listing=a1,
                                amount=600, is_active=True, is_winner=False)

    def test_listing_counts(self):
        """ Check that listing count is correct """
        u = User.objects.get(username="Ronald")
        self.assertEqual(u.auctionlisting_set.count(), 2)

    def test_is_valid_listing(self):
        """ Check that valid listing is valid """
        a = AuctionListing.objects.get(item="1964 Mustang")
        self.assertTrue(a.is_valid_listing())

    def test_invalid_expiration_listing(self):
        a = AuctionListing.objects.get(item="Lego Camper")
        self.assertFalse(a.is_valid_listing())

    def test_invalid_buyout_listing(self):
        """ Check that invalid buyout price is invalid """
        a = AuctionListing.objects.get(item="Superman Action Figure")
        self.assertFalse(a.is_valid_listing())

    def test_invalid_starting_bid_listing(self):
        """ Check that invalid starting bid is invalid """
        a = AuctionListing.objects.get(item="")
        self.assertFalse(a.is_valid_listing())

    def test_valid_bid(self):
        """ Check that valid bid is valid """
        b = Bid.objects.get(amount=300)
        self.assertTrue(b.is_valid_bid())

    def test_invalid_double_bid(self):
        """ Check that double active bid on same item is invalid """
        a = AuctionListing.objects.get(item="1964 Mustang")
        u = User.objects.get(username="Dennis")
        bids = Bid.objects.filter(bidder=u, listing=a, is_active=True)
        self.assertGreater(bids.count(), 1)

    def test_index(self):
        """ Check that index returns all open listings """
        c = Client()
        response = c.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response.context["listings_with_bids"]), 4)

    def test_valid_listing_page(self):
        """ Check that valid listing returns status code of 200 """
        a = AuctionListing.objects.get(item="1964 Mustang")

        c = Client()
        response = c.get(f"/listing/{a.id}")
        self.assertEqual(response.status_code, 200)

    def test_invalid_listing_page(self):
        """ Check that listing page for non-existant listing is invalid """
        max_id = AuctionListing.objects.all().aggregate(Max("id"))["id__max"]

        c = Client()
        response = c.get(f"/listing/{max_id + 1}")
        self.assertEqual(response.status_code, 404)

    def test_listing_page_bid(self):
        """ Check that listing page returns current price """
        a = AuctionListing.objects.get(item="1964 Mustang")

        c = Client()
        response = c.get(f"/listing/{a.id}")
        self.assertIsNotNone(response.context["bid_amount"])
`

#### Browser Testing
- [Selenium](#selenium)

### Integration Testing
