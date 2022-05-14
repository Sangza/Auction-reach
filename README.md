'reach 0.1';

const Autioneeprops = Object({
    initialbid: UInt,
    timeout:UInt;
})

const Biderprops = {
    getBid: Fun([UInt], Maybe(UInt))
}

const OwnerInterface = {
    showOwner: Fun([UInt, Address], Null)
    getAuctionProps: Fun([],AuctioneeProps)
    ...Biderprops
}

const CreatorInterface = {
    ...OwnerInterface,
    getId: Fun([],UInt)

}

const emptyAuction = {initialbid: 0, timeout: 0}

export const main = Reach.Ap(() => {

    const Creator = Participant('Creator', CreatorInterface);
    const Owner = ParticipantClass('Owner', OwnerInterface);
    init();

    Creator.only =(() =>{
         id: declassify(interact.getId())
    })
    Creator.publish(id);

    var Owner = Creator
    invariant(balance() == 0);
   while (true) {
       commit();

    Owner.only =(() => {
        interact.showOwner(id, owner);
        const amOwner == this == owner
        const {initialbid, timeout} =
        amOwner ? declassify(interact.getAutioneepage()) :emptyAuction
    });
    Owner
      .publish(initialbid, timeout)
      .when(amOwner)
      .timeout(false);

      const[timeRemaining, keepGoing] = makeDeadline(timeout)

      const[ winner, isFirstBid, currentPrice] =
       parallelReduce([owner, true, initialbid])
        .invariant(balance() == (isFirstBid ? 0 : currentPrice))
        .while(keepGoing())
        .case(Owner,
           () => {
             const mbid = (this! == owner && this! == winner)
             ? declassify(interact.getBid(currentPrice))
             : Maybe(UInt).None();
            return ({
                when: maybe(mbid, false, (bid) => (bid > currentPrice))
                msg: fromSome(mbid, 0)
            });
           }
           (bid) => bid,
           (bid) => {
               require(bid > currentPrice);
            transfer(isFirstBid ? 0 : currentPrice).to(winner);
            return[ this, false,bid ];
           }
           
        )
        .timeRemaining(timeRemaining());

    transfer(isFirstBid ? 0 : currentPrice).to(owner);

    owner = winner;
    continue;

   };
   
   commit();
   exit();

});
