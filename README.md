@extends('layouts.app')

@section('content')
<div class="d-flex justify-content-between mb-3">
    <h2>Card List</h2>
    <a class="btn btn-primary" href="{{ route('cards.create') }}">Create Card</a>
</div>

@if (session('success'))
    <div class="alert alert-success">{{ session('success') }}</div>
@endif

<table class="table table-bordered">
    <tr>
        <th>ID</th>
        <th>Title</th>
        <th>Description</th>
        <th>Image</th>
        <th>Action</th>
    </tr>
    @foreach ($cards as $card)
    <tr data-character="{{ $card->character ?? '' }}"> <!-- adiciona data-character -->
        <td>{{ $card->id }}</td>
        <td>{{ $card->title }}</td>
        <td>{{ $card->description }}</td>
        <td> 
            @if($card->image)
                <img src="{{ asset('storage/' . $card->image) }}" alt="{{ $card->title }}" width="100">
            @endif
        </td>
        <td>
            <a class="btn btn-info btn-sm" href="{{ route('cards.show', $card) }}">Show</a>
            <a class="btn btn-warning btn-sm" href="{{ route('cards.edit', $card) }}">Edit</a>
            <form action="{{ route('cards.destroy', $card) }}" method="POST" class="d-inline">
                @csrf @method('DELETE')
                <button type="submit" class="btn btn-danger btn-sm">Delete</button>
            </form>
        </td>
    </tr>
    @endforeach
</table>

{{ $cards->links() }}
@endsection

@push('scripts')
<script>
  const voiceLines = {
    duke: [
      'duke_main_1.mp3',
      'duke_main_2.mp3',
      'duke_main_3.mp3',
      'duke_main_4.mp3',
      'duke_main_5.mp3'
    ],
    assassin: [
      'assassin_main_1.mp3',
      'assassin_main_2.mp3',
      'assassin_main_3.mp3',
      'assassin_main_4.mp3',
      'assassin_main_5.mp3'
    ],
    captain: [
      'captain_main_1.mp3',
      'captain_main_2.mp3',
      'captain_main_3.mp3',
      'captain_main_4.mp3',
      'captain_main_5.mp3'
    ],
    ambassador: [
      'ambassador_main_1.mp3',
      'ambassador_main_2.mp3',
      'ambassador_main_3.mp3',
      'ambassador_main_4.mp3',
      'ambassador_main_5.mp3'
    ],
    contessa: [
      'contessa_main_1.mp3',
      'contessa_main_2.mp3',
      'contessa_main_3.mp3',
      'contessa_main_4.mp3',
      'contessa_main_5.mp3'
    ]
  };

  document.querySelectorAll('tr[data-character]').forEach(row => {
    row.addEventListener('click', () => {
      const character = row.dataset.character;
      if (!character) return;

      const audios = voiceLines[character.toLowerCase()];
      if (!audios) return;

      const randomIndex = Math.floor(Math.random() * audios.length);
      const audioSrc = '/audios/' + audios[randomIndex];

      const audio = new Audio(audioSrc);
      audio.play();
    });
  });
</script>
@endpush
