/**
 * Alexa Skill Kit: Deals Now
 * @author: shengbi
 */
'use strict';

const Alexa = require('alexa-sdk');
const APP_ID = '<your_application_id>'; 

const HTTPS = require('https');

const handlers = {
    'NewSession': function () {
        this.attributes.speechOutput = this.t('WELCOME_MESSAGE', this.t('SKILL_NAME'));
        // If the user either does not reply to the welcome message or says something that is not
        // understood, they will be prompted again with this text.
        this.attributes.repromptSpeech = this.t('WELCOME_REPROMT');
        this.emit(':ask', this.attributes.speechOutput, this.attributes.repromptSpeech);
    },
    'PopularDealsNow': function () {
        getPopularDeals(this);
    },
    'AMAZON.HelpIntent': function () {
        this.attributes.speechOutput = this.t('HELP_MESSAGE');
        this.attributes.repromptSpeech = this.t('HELP_REPROMT');
        this.emit(':ask', this.attributes.speechOutput, this.attributes.repromptSpeech);
    },
    'AMAZON.RepeatIntent': function () {
        this.emit(':ask', this.attributes.speechOutput, this.attributes.repromptSpeech);
    },
    'AMAZON.StopIntent': function () {
        this.emit('SessionEndedRequest');
    },
    'AMAZON.CancelIntent': function () {
        this.emit('SessionEndedRequest');
    },
    'SessionEndedRequest': function () {
        this.emit(':tell', this.t('STOP_MESSAGE'));
    },
};

const languageStrings = {
    'en-US': {
        translation: {
            SKILL_NAME: "Deals Now",
            WELCOME_MESSAGE: "Welcome to Deals Now.",
            WELCOME_REPROMT: "You can listen to deals news by asking Alexa: What are the popular deals now?",
            DISPLAY_CARD_TITLE: "Deals Now",
            HELP_MESSAGE: "You can listen to deals news by asking Alexa: What are the popular deals now?",
            HELP_REPROMT: "You can listen to deals news by asking Alexa: What are the popular deals now?",
            STOP_MESSAGE: "Goodbye!"
        },
    }
};

exports.handler = (event, context) => {
    const alexa = Alexa.handler(event, context);
    alexa.appId = APP_ID;
    // To enable string internationalization (i18n) features, set a resources object.
    alexa.resources = languageStrings;
    alexa.registerHandlers(handlers);
    alexa.execute();
};





// --------------- Data functions ----------------

function getPopularDeals(handler) {
    HTTPS.get("https://slickdeals.net/newsearch.php?mode=frontpage&searcharea=deals&searchin=first&rss=1", function(res) {
        res.setEncoding('utf8');
        var body = '';
        res.on('data', function(d) {
            body += d;
        });
        res.on('end', function() {
            try {
                var regex = /<title>.*<\/title>/gm;
                var items = body.match(regex);
                if (items !== null) {
                    items = items.map(function(it) { return it.slice(7, -8); });
                    items = items.map(function(it) { 
                        if (it.match(/^<!\[CDATA\[.*\]\]>$/) !== null) {
                            return it.slice(9, -3);
                        }
                        return it;
                    });
                    items = items.filter(function (item) {
                        if (!item.match(/Slickdeals/i)) {
                            return item;
                        }
                    });
                    var deals = getRandomTopDeals(items);
                    var speech = getDealsToSpeech(deals);
                    handler.emit(':tell', speech);
                }
                
            } catch (err) {
                console.error('Unable to parse response', err);
            }
        });
    }).on('error', function(err) {
        console.error('Error with the request:', err.message);
    });
}

/**
 * Returns deals speech
 */
function getDealsToSpeech(deals) {

    var speech = "";
    for (var i = 0; i < deals.length; i++) {
        var deal = deals[i];
        try {
            // Remove () and the content inside
            var openIndex;
            while ((openIndex = deal.indexOf("(")) > -1) {
                var closeIndex = deal.indexOf(")");
                deal = deal.substr(0, openIndex - 1) + deal.substr(closeIndex + 1);
            }
            // Replace w+ or w/ with with
            deal = deal.replace(/w[+/]/gi, " with ");
            // Replace S&H with shipping and handling 
            deal = deal.replace(/\sS\&H[\s]{0,1}/g, " shipping and handling ");
            // Replace AC with after coupon
            deal = deal.replace(/\sAC[\s,.;]{0,1}/g, " after coupon ");
            // Replace AR with after rebate
            deal = deal.replace(/\sAR[\s,.;]{0,1}/g, " after rebate ");
            // Replace FS with free shipping
            deal = deal.replace(/\sFS[\s,.;]{0,1}/g, " free shipping ");
            // Replace FSSS with free super saver shipping
            deal = deal.replace(/\sFSSS[\s,.;]{0,1}/g, " free super saver shipping ");
            // Replace S&S with subscribe and save
            deal = deal.replace(/\sS\&S[\s,.;]{0,1}/g, " subscribe and save ");
            // Replace "' with inch
            deal = deal.replace(/["']/gi, " inch ");
            // Replace + with plus
            deal = deal.replace(/\+/gi, " plus ");
            // Replace @ with at
            deal = deal.replace(/\@/gi, " at ");
            // Replace & with and
            deal = deal.replace(/\&/gi, " and ");
            // Remove -, :, *
            deal = deal.replace(/[-:*]/gi, " ");
            
            // Remove extra blank space
            deal = deal.replace(/[ ]{2,}/gi, " ");
            speech += deal + "<break time='1s'/>";
            //console.log(deal);
        } catch (err) {
            console.error('Error while generating speech friendly deal description', err);
        }
    }
    return speech;
}

/**
 * Returns a list of randomly selected deals of size TOP_COUNT
 */
function getRandomTopDeals(deals) {
    
    var TOP_COUNT = 5;
    var pickedIndex = [];
    var randomDeals = [];
    var dealsCount = deals.length;
    for (var i = 0; i < TOP_COUNT; i++) {
        var rand = getRandomInt(0, dealsCount - 1);
        if (pickedIndex.indexOf(rand) === -1) {
            pickedIndex.push(rand);
            randomDeals.push(deals[rand]);
        } else {
            i--;
        }
    }
    return randomDeals;
}

/**
 * Returns a random integer between min (inclusive) and max (inclusive)
 */
function getRandomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
