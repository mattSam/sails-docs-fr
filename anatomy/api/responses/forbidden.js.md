# api/responses/forbidden.js
### Objectif

Il s'agit de l'une des réponses serveur par défaut incluses dans un nouveau projet Sails.

Il contient la logique de ce qui devrait se produire lorsque vous souhaitez émettre une réponse http 403. La réponse peut être émise à partir de n'importe quel endroit où vous avez accès à l'objet express `res` en appelant res.forbidden.

Feel free to edit this file to suit your needs.  You can also create a custom response using our `sails-generate-custom-response` generator.

Pour plus d'informations, reportez-vous à la section Réponse de la documentation de référence.


<docmeta name="displayName" value="forbidden.js">

```
/**
 * 403 (Forbidden) Handler
 *
 * Usage:
 * return res.forbidden();
 * return res.forbidden(err);
 * return res.forbidden(err, view);
 * return res.forbidden(err, redirectTo);
 *
 * e.g.:
 * return res.forbidden('Access denied.');
 *
 */

module.exports = function forbidden (err, viewOrRedirect) {

  // Get access to `req` & `res`
  var req = this.req;
  var res = this.res;

  // Serve JSON (with optional JSONP support)
  function sendJSON (data) {
    if (!data) {
      return res.send();
    }
    else {
      if (typeof data !== 'object' || data instanceof Error) {
        data = {error: data};
      }
      if ( req.options.jsonp && !req.isSocket ) {
        return res.jsonp(data);
      }
      else return res.json(data);
    }
  }

  // Set status code
  res.status(403);

  // Log error to console
  this.req._sails.log.verbose('Sent 403 ("Forbidden") response');
  if (err) {
    this.req._sails.log.verbose(err);
  }

  // If the user-agent wants JSON, always respond with JSON
  if (req.wantsJSON) {
    return sendJSON(err);
  }

  // Make data more readable for view locals
  var locals;
  if (!err) { locals = {}; }
  else if (typeof err !== 'object'){
    locals = {error: err};
  }
  else {
    var readabilify = function (value) {
      if (sails.util.isArray(value)) {
        return sails.util.map(value, readabilify);
      }
      else if (sails.util.isPlainObject(value)) {
        return sails.util.inspect(value);
      }
      else return value;
    };
    locals = { error: readabilify(err) };
  }

  // Serve HTML view or redirect to specified URL
  if (typeof viewOrRedirect === 'string') {
    if (viewOrRedirect.match(/^(\/|http:\/\/|https:\/\/)/)) {
      return res.redirect(viewOrRedirect);
    }
    else return res.view(viewOrRedirect, locals, function viewReady(viewErr, html) {
      if (viewErr) return sendJSON(err);
      else return res.send(html);
    });
  }
  else return res.view('403', locals, function viewReady(viewErr, html) {
    if (viewErr) return sendJSON(err);
    else return res.send(html);
  });
};

```
